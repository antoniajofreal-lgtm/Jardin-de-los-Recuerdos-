<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1.0" />
<title>El Jardín de los Recuerdos 🌳</title>
<style>
  :root { --bottom-space: 200px; }
  html,body{
    height:100%; margin:0;
    font-family: "Comic Sans MS", cursive, system-ui, -apple-system, "Segoe UI", Roboto, Arial;
    /* fondo estable: Wikimedia Commons (dominio público) */
    background: url("https://upload.wikimedia.org/wikipedia/commons/6/6e/Field_in_summer.JPG") center/cover no-repeat fixed;
    background-size: cover;
    text-align: center;
    color: #2c2c2c;
  }

  h1 { margin-top: 20px; color:#fff; text-shadow:2px 2px 4px #000; }
  #gameArea { margin:20px auto; max-width:860px; padding:12px; }

  /* Grid adaptable */
  #grid {
    display:grid;
    justify-content:center;
    gap:12px;
    margin:20px auto;
  }

  .cell {
    width:84px; height:84px;
    display:flex; align-items:center; justify-content:center;
    font-size:40px; cursor:pointer;
    background:rgba(255,255,255,0.85); border-radius:12px;
    transition:transform 0.18s, background 0.18s;
    touch-action: manipulation; /* reduce zoom/latency on mobile */
    -webkit-tap-highlight-color: rgba(0,0,0,0);
    user-select:none;
  }
  .cell.active { background:#ffe066; transform:scale(1.08); }

  #gardener { position:relative; display:inline-block; margin-top:10px; vertical-align:top; }
  #gardener img { width:120px; height:auto; display:block; }
  #speech {
    position:absolute; top:-60px; left:130px;
    background:#fff; padding:10px 15px; border-radius:12px; border:1px solid rgba(0,0,0,0.12);
    max-width:260px; text-align:left; box-shadow:0 6px 18px rgba(0,0,0,0.18);
    font-size:15px;
  }

  #lives, #score { font-size:18px; margin:10px; color:#fff; text-shadow:1px 1px 2px #000; display:inline-block; vertical-align:middle; }

  /* final screen */
  #finalScreen {
    position:fixed;top:0;left:0;right:0;bottom:0;
    background:rgba(0,0,0,0.75); color:#fff; display:none;
    align-items:center; justify-content:center; flex-direction:column; gap:12px; padding:20px;
    z-index:2000;
  }

  /* vida parpadeo para notificar pérdida */
  .blinkLife {
    animation: blinkLifeAnim 0.9s ease-in-out 2;
  }
  @keyframes blinkLifeAnim {
    0% { opacity: 1; transform: translateY(0); color: inherit; }
    25% { opacity: 0.2; transform: translateY(-2px); color: #ff5252; }
    50% { opacity: 1; transform: translateY(0); color: inherit; }
    75% { opacity: 0.2; transform: translateY(-2px); color: #ff5252; }
    100% { opacity: 1; transform: translateY(0); color: inherit; }
  }

  button {
    font-size:16px; padding:10px 18px; border:none; border-radius:8px; cursor:pointer;
    background:#4CAF50; color:white;
  }

  @media(max-width:520px){
    .cell{ width:72px; height:72px; font-size:34px; }
    #speech{ left:100px; top:-56px; font-size:14px; max-width:180px; }
    :root{ --bottom-space: 170px; }
  }
</style>
</head>
<body>
  <h1>🌳 El Jardín de los Recuerdos 🌳</h1>

  <div id="gameArea">
    <div id="gardener">
      <img id="gardenerImg" src="https://cdn.pixabay.com/photo/2021/02/18/19/35/gardener-6027144_1280.png" alt="Jardinero">
      <div id="speech">¡Bienvenido al jardín!</div>
    </div>

    <div style="display:inline-block; margin-left:18px; vertical-align:middle">
      <div id="lives"></div>
      <div id="score">Puntos: 0</div>
    </div>

    <div id="grid" role="application" aria-label="Tablero de juego"></div>

    <div style="margin-top:12px;">
      <button id="startBtn">Iniciar Juego</button>
      <button id="musicBtn">🔊 Música</button>
    </div>
  </div>

  <div id="finalScreen">
    <h2 id="finalTitle"></h2>
    <div id="finalStats"></div>
    <button id="playAgainBtn">Jugar de nuevo</button>
  </div>

  <!-- sonidos -->
  <audio id="sndClick" src="https://actions.google.com/sounds/v1/cartoon/wood_plank_flicks.ogg"></audio>
  <audio id="sndErr" src="https://actions.google.com/sounds/v1/cartoon/metal_thud_and_clang.ogg"></audio>
  <audio id="bgMusic" src="https://files.freemusicarchive.org/storage-freemusicarchive-org/music/ccCommunity/Jahzzar/Traveller/Jahzzar_-_05_-_Siesta.mp3" loop></audio>

<script>
/* ----- Refresco: fallback emoji image if gardener fails ----- */
function emojiDataURI(emoji, size=160){
  const svg = `<svg xmlns='http://www.w3.org/2000/svg' width='${size}' height='${size}'>
    <rect width='100%' height='100%' fill='%23ffffff'/>
    <text x='50%' y='50%' dominant-baseline='middle' text-anchor='middle' font-size='72'>${emoji}</text>
  </svg>`;
  return 'data:image/svg+xml;utf8,' + encodeURIComponent(svg);
}
const gardenerImg = document.getElementById('gardenerImg');
gardenerImg.addEventListener('error', ()=> { gardenerImg.src = emojiDataURI('🧑‍🌾', 160); });

/* ----- DOM refs ----- */
const grid = document.getElementById('grid');
const speech = document.getElementById('speech');
const livesDiv = document.getElementById('lives');
const scoreDiv = document.getElementById('score');
const startBtn = document.getElementById('startBtn');
const musicBtn = document.getElementById('musicBtn');
const finalScreen = document.getElementById('finalScreen');
const finalTitle = document.getElementById('finalTitle');
const finalStats = document.getElementById('finalStats');
const playAgainBtn = document.getElementById('playAgainBtn');

const sndClick = document.getElementById('sndClick');
const sndErr = document.getElementById('sndErr');
const bgMusic = document.getElementById('bgMusic');

/* ----- volumes ----- */
sndClick.volume = 0.6;
sndErr.volume = 1.0;

/* ----- Game configuration ----- */
const fruits = ["🍎","🍐","🍊","🍇","🍓","🍒","🍉","🥕","🍍","🥝","🥦","🍅"];
const roundNames = ["Sembrar las frutas","Cosechar las frutas","A juntar la cosecha"];
const roundCells = [4,6,8]; // cells per round
const SEQUENCES_PER_ROUND = 2;

/* ----- State ----- */
let currentRound = 1;
let boardSize = 4;
let sequence = [];
let playerIndex = 0;
let sequenceCount = 0;
let score = 0;
let lives = 3;
let listening = false;
let musicOn = false;
let correctMoves = 0;
let errors = 0;

/* ----- Utilities ----- */
function safePlay(audioEl){
  if(!audioEl) return;
  try{
    audioEl.pause();
    audioEl.currentTime = 0;
    audioEl.play().catch(()=>{ /* allowed to fail silently if browser blocks */ });
  } catch(e){ console.debug('safePlay err', e); }
}

function showSpeech(msg, ms=3000){
  speech.textContent = msg;
  speech.style.display = 'block';
  if(ms>0){
    setTimeout(()=>{ if(speech.textContent === msg) speech.style.display = 'none'; }, ms);
  }
}

function updateHUD(){
  scoreDiv.textContent = 'Puntos: ' + score;
  // show lives as flowers, update innerHTML to allow class animation
  let html = 'Vidas: ';
  for(let i=0;i<3;i++){
    if(i < lives) html += '🌸';
    else html += '<span style="opacity:0.4">🌸</span>';
  }
  livesDiv.innerHTML = html;
}

/* ----- Build board for round ----- */
function buildBoard(round){
  grid.innerHTML = '';
  const n = roundCells[round-1];
  // set grid columns: up to 4 columns for layout; it will wrap rows automatically
  const cols = Math.min(4, n);
  grid.style.gridTemplateColumns = `repeat(${cols}, 84px)`;
  // choose first n icons from fruits (stable order) — you can randomize if you prefer
  for(let i=0;i<n;i++){
    const cell = document.createElement('div');
    cell.className = 'cell';
    cell.dataset.idx = String(i);
    cell.textContent = fruits[i % fruits.length];
    // use pointerdown to reduce mobile delay, prevent default to avoid weird zoom
    cell.addEventListener('pointerdown', function(ev){
      ev.preventDefault();
      onCellPressed(i);
    });
    grid.appendChild(cell);
  }
}

/* ----- Sequence generation and display ----- */
function generateSequence(len){
  const poolLen = roundCells[currentRound-1];
  const seq = [];
  for(let i=0;i<len;i++) seq.push(Math.floor(Math.random()*poolLen));
  return seq;
}

async function showSequence(seq){
  listening = false;
  showSpeech('Observa la secuencia...', 2200);
  await new Promise(r=>setTimeout(r, 2200));
  const cells = Array.from(grid.querySelectorAll('.cell'));
  for(let i=0;i<seq.length;i++){
    const idx = seq[i];
    const cell = cells[idx];
    if(cell){
      cell.classList.add('active');
      safePlay(sndClick); // same sound used for show and player correct
      await new Promise(r=>setTimeout(r, 700));
      cell.classList.remove('active');
      await new Promise(r=>setTimeout(r, 150));
    } else {
      await new Promise(r=>setTimeout(r, 300));
    }
  }
  listening = true;
  showSpeech('¡Tu turno! Repite la secuencia', 2200);
}

/* ----- Round flow ----- */
function startRound(){
  if(currentRound > roundCells.length){
    finishGame(true);
    return;
  }
  sequenceCount = 0;
  showSpeech(`Ronda ${currentRound}: ${roundNames[currentRound-1]}`, 4000);
  buildBoard(currentRound);
  // give a little time to read
  setTimeout(()=> startNewSequence(), 1200);
}

function startNewSequence(){
  // length grows slightly with round and sequence count
  const len = 2 + (currentRound - 1) + sequenceCount;
  sequence = generateSequence(len);
  playerIndex = 0;
  showSequence(sequence).catch(e=>console.debug(e));
}

/* ----- Life lost notification ----- */
function notifyLifeLost(){
  // play error sound loudly first
  safePlay(sndErr);
  // visually highlight lives area: add class, remove later
  livesDiv.classList.add('blinkLife');
  setTimeout(()=> livesDiv.classList.remove('blinkLife'), 1000);
  // gardener bubble message
  showSpeech(`¡Oh no! Perdimos una vida. Te quedan ${lives} 🌸`, 3200);
}

/* ----- Player interaction ----- */
function onCellPressed(idx){
  if(!listening) return;
  // if incorrect:
  if(idx !== sequence[playerIndex]){
    errors++;
    listening = false;
    playerIndex = 0;
    // decrease life, notify
    lives = Math.max(0, lives - 1);
    updateHUD();
    notifyLifeLost();
    if(lives <= 0){
      setTimeout(()=> finishGame(false), 800);
    } else {
      // repeat same sequence after short delay
      setTimeout(()=> showSequence(sequence), 1000);
    }
    return;
  }

  // correct press
  safePlay(sndClick);
  // visual feedback
  const cell = grid.querySelectorAll('.cell')[idx];
  if(cell){ cell.classList.add('active'); setTimeout(()=>cell.classList.remove('active'),220); }
  playerIndex++;
  score += 10;
  correctMoves++;
  updateHUD();

  if(playerIndex >= sequence.length){
    // sequence completed
    score += 20;
    updateHUD();
    sequenceCount++;
    listening = false;
    if(sequenceCount < SEQUENCES_PER_ROUND){
      showSpeech('¡Muy bien! Ahora otra secuencia.', 1800);
      setTimeout(()=> startNewSequence(), 1000);
    } else {
      showSpeech('¡Ronda completada! 🎉', 1800);
      currentRound++;
      setTimeout(()=> startRound(), 1200);
    }
  }
}

/* ----- Finish game and stats ----- */
function finishGame(won){
  listening = false;
  // hide main UI, show final overlay
  finalScreen.style.display = 'flex';
  if(won){
    finalTitle.textContent = '🌟 ¡Gracias por jugar! ¡Tu jardín florece! 🌸';
  } else {
    finalTitle.textContent = '💪 ¡Ánimo! Intentemos de nuevo';
  }
  finalStats.innerHTML = `
    Puntos: <strong>${score}</strong><br>
    Rondas completadas: <strong>${Math.min(currentRound-1, roundCells.length)}</strong><br>
    Aciertos (pasos correctos): <strong>${correctMoves}</strong><br>
    Errores: <strong>${errors}</strong>
  `;
  // stop music
  try{ bgMusic.pause(); } catch(e){}
}

/* ----- Controls: start / music / replay ----- */
startBtn.addEventListener('click', ()=>{
  // reset and start
  finalScreen.style.display = 'none';
  currentRound = 1; score = 0; lives = 3; sequenceCount = 0; playerIndex = 0;
  sequence = []; listening = false; correctMoves = 0; errors = 0;
  updateHUD();
  // try autoplaying music (may be blocked until user interacts)
  safePlay(bgMusic);
  musicOn = !bgMusic.paused;
  startRound();
});

musicBtn.addEventListener('click', ()=>{
  if(musicOn){
    try{ bgMusic.pause(); }catch(e){}
    musicOn = false;
    musicBtn.textContent = '🔊 Música';
  } else {
    safePlay(bgMusic);
    musicOn = !bgMusic.paused;
    musicBtn.textContent = musicOn ? '🔇 Música' : '🔊 Música';
  }
});

playAgainBtn.addEventListener('click', ()=>{
  finalScreen.style.display = 'none';
  // simulate start click
  startBtn.click();
});

/* ----- init HUD ----- */
updateHUD();
showSpeech('¡Hola! Presiona "Iniciar Juego".', 4000);

</script>
</body>
</html>
