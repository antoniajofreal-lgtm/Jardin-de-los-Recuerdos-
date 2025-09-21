<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1.0" />
<title>El Jardín de los Recuerdos 🌳</title>
<style>
  :root {
    --bottom-space: 200px; /* espacio para jardinero */
  }
  html,body{
    height:100%;
    margin:0;
    font-family: system-ui, -apple-system, "Segoe UI", Roboto, Arial;
    background: url("https://cdn.pixabay.com/photo/2015/06/19/21/24/avenue-815297_1280.jpg") center/cover no-repeat fixed;
    color: #153e2e;
  }

  /* START SCREEN */
  #startScreen {
    position:fixed; inset:0;
    display:flex; flex-direction:column; align-items:center; justify-content:center;
    background: rgba(255,255,255,0.86);
    z-index: 900;
    padding:20px; text-align:center;
  }
  #startScreen h1 { margin:0 0 8px; font-size:2.1rem; color:#1f6b34; }
  #startScreen p { color:#2b6a36; max-width:640px; }

  .btn {
    padding:10px 18px; border-radius:10px; border:0; cursor:pointer;
    background:#57b86a; color:white; font-size:1rem; margin:8px;
  }
  .btn.alt { background:#6ca3f5; }

  /* GAME */
  #gameContainer { display:none; padding:18px; min-height:100vh; box-sizing:border-box; }
  header.appbar { display:flex; justify-content:center; margin-bottom:8px; }
  header.appbar h2 { margin:0; color:#143e29; background: rgba(255,255,255,0.7); padding:6px 14px; border-radius:8px; }

  .hud { display:flex; justify-content:space-between; align-items:center; gap:12px; margin-bottom:10px; color:#1e6b2a; font-size:18px; }
  #elements {
    display:grid;
    grid-template-columns: repeat(4, minmax(72px, 84px));
    gap:14px;
    justify-content:center;
    padding-bottom: var(--bottom-space);
    margin-top:6px;
  }
  .cell {
    width:84px; height:84px; border-radius:12px; background: rgba(255,255,255,0.95);
    display:flex; align-items:center; justify-content:center; font-size:40px;
    box-shadow:0 6px 14px rgba(0,0,0,0.12);
    touch-action: manipulation; user-select:none;
  }
  .cell.active { transform: scale(1.08); background:#c7f1d6; transition: all .16s; }

  /* Jardinero */
  #gardenerWrapper {
    position:fixed; right:12px; bottom:12px; display:flex; flex-direction:column; align-items:center;
    gap:10px; z-index:1000; pointer-events:none;
  }
  #speechBubble {
    min-width:160px; max-width:260px; font-size:16px; background: rgba(255,255,255,0.98);
    color:#173d2a; border-radius:12px; padding:10px; border:1px solid rgba(0,0,0,0.08);
    box-shadow:0 10px 26px rgba(0,0,0,0.12);
    text-align:center;
  }
  #gardenerImg { width:110px; display:block; }

  /* overlays */
  .overlay { position:fixed; inset:0; display:none; align-items:center; justify-content:center; z-index:1200; background: rgba(255,255,255,0.95); }
  .overlay h2 { color:#1f6b34; margin:0 0 8px; }

  /* Animación de pérdida de vida */
  .lifeLost {
    animation: blinkRed 0.8s ease-in-out 2;
  }
  @keyframes blinkRed {
    0%,100% { color:#1e6b2a; }
    50% { color:#c62828; }
  }

  @media(max-width:520px){
    .cell{ width:72px; height:72px; font-size:34px; }
    #speechBubble{ font-size:15px; max-width:200px; }
    :root { --bottom-space: 170px; }
  }
</style>
</head>
<body>

  <!-- Pantalla inicio -->
  <div id="startScreen">
    <h1>🌳 El Jardín de los Recuerdos</h1>
    <p>Ayuda al jardinero a cuidar su jardín recordando secuencias. 2 secuencias por ronda — sin penalizaciones, 3 vidas.</p>
    <div style="margin-top:14px;">
      <button id="startBtn" class="btn">🌸 Comenzar Juego</button>
      <button id="musicBtnStart" class="btn alt">🔊 Música</button>
    </div>
  </div>

  <!-- Juego -->
  <div id="gameContainer">
    <header class="appbar"><h2>El Jardín de los Recuerdos</h2></header>
    <div class="hud">
      <div id="scoreDisplay">Puntos: 0</div>
      <div id="livesDisplay">Vidas: 🌸🌸🌸</div>
    </div>

    <main id="elements"></main>

    <div style="display:flex; justify-content:center; gap:8px; margin-top:10px;">
      <button id="restartBtn" class="btn" style="background:#f6a45d">🔄 Reiniciar</button>
      <button id="musicBtnGame" class="btn alt">🔊 Música</button>
    </div>
  </div>

  <!-- Jardinero -->
  <div id="gardenerWrapper">
    <div id="speechBubble">¡Hola! Presiona "Comenzar Juego".</div>
    <img id="gardenerImg" src="https://cdn.pixabay.com/photo/2014/04/03/11/53/gardener-311325_1280.png" alt="Jardinero">
  </div>

  <!-- Overlays -->
  <div id="winOverlay" class="overlay"><div><h2>🌟 ¡Gracias por jugar! 🌟</h2><p>El jardín te agradece.</p><div style="margin-top:12px;"><button id="winRestart" class="btn">Volver a jugar</button></div></div></div>
  <div id="loseOverlay" class="overlay"><div><h2>💪 ¡Ánimo! 💪</h2><p>Inténtalo otra vez.</p><div style="margin-top:12px;"><button id="loseRestart" class="btn">Reintentar</button></div></div></div>

  <!-- Audios -->
  <audio id="bgMusic" loop src="https://files.freemusicarchive.org/storage-freemusicarchive-org/music/ccCommunity/Jahzzar/Traveller/Jahzzar_-_05_-_Siesta.mp3"></audio>
  <audio id="sndSeq" src="https://actions.google.com/sounds/v1/cartoon/wood_plank_flicks.ogg"></audio>
  <audio id="sndOk" src="https://actions.google.com/sounds/v1/cartoon/wood_plank_flicks.ogg"></audio>
  <audio id="sndErr" src="https://actions.google.com/sounds/v1/cartoon/metal_thud_and_clang.ogg"></audio>

<script>
/* ---------- Fallback emoji si falla la imagen ---------- */
function emojiDataURI(emoji, size=160){
  const svg = `<svg xmlns='http://www.w3.org/2000/svg' width='${size}' height='${size}'><rect width='100%' height='100%' fill='%23ffffff'/><text x='50%' y='50%' dominant-baseline='middle' text-anchor='middle' font-size='72'>${emoji}</text></svg>`;
  return 'data:image/svg+xml;utf8,' + encodeURIComponent(svg);
}
const gardenerImg = document.getElementById('gardenerImg');
gardenerImg.addEventListener('error', ()=> { gardenerImg.src = emojiDataURI('🧑‍🌾', 160); });

/* ---------- DOM refs ---------- */
const startScreen = document.getElementById('startScreen');
const startBtn = document.getElementById('startBtn');
const musicBtnStart = document.getElementById('musicBtnStart');
const gameContainer = document.getElementById('gameContainer');
const elementsWrap = document.getElementById('elements');
const scoreDisplay = document.getElementById('scoreDisplay');
const livesDisplay = document.getElementById('livesDisplay');
const restartBtn = document.getElementById('restartBtn');
const musicBtnGame = document.getElementById('musicBtnGame');
const speechBubble = document.getElementById('speechBubble');
const winOverlay = document.getElementById('winOverlay');
const loseOverlay = document.getElementById('loseOverlay');
const winRestart = document.getElementById('winRestart');
const loseRestart = document.getElementById('loseRestart');

const bgMusic = document.getElementById('bgMusic');
const sndSeq = document.getElementById('sndSeq');
const sndOk = document.getElementById('sndOk');
const sndErr = document.getElementById('sndErr');

/* ---------- Estado ---------- */
const pools = [
  ['🌸','🌻','🌷','🌼'], 
  ['🍎','🍌','🍐','🍇','🍉','🍊'], 
  ['🥕','🌽','🍅','🍆','🥦','🥒','🍄','🥬']
];

let currentRound = 1;
let sequence = [];
let playerIndex = 0;
let sequenceCount = 0;
let score = 0;
let lives = 3;
let listening = false;
let musicOn = false;

/* ---------- Helpers ---------- */
function setBubble(text, ms=3000){
  speechBubble.textContent = text;
  speechBubble.style.display = 'block';
  setTimeout(()=>{ if(speechBubble.textContent === text) speechBubble.style.display = 'none'; }, ms);
}
function updateHUD(){
  scoreDisplay.textContent = 'Puntos: ' + score;
  livesDisplay.textContent = 'Vidas: ' + '🌸'.repeat(Math.max(0,lives));
}
async function safePlay(audioEl){
  if(!audioEl) return;
  try{ audioEl.currentTime = 0; await audioEl.play(); } catch(e){ console.debug('audio blocked', e); }
}

/* ---------- Música ---------- */
async function toggleMusic(){
  if(musicOn){ bgMusic.pause(); musicOn=false; }
  else { await safePlay(bgMusic); musicOn=!bgMusic.paused; }
  const label = musicOn ? '🔇 Música' : '🔊 Música';
  musicBtnStart.textContent = label;
  musicBtnGame.textContent = label;
}
musicBtnStart.addEventListener('click', toggleMusic);
musicBtnGame.addEventListener('click', toggleMusic);

/* ---------- Tablero ---------- */
function buildBoard(roundIndex){
  elementsWrap.innerHTML = '';
  const pool = pools[roundIndex - 1];
  pool.forEach((symbol, idx)=>{
    const div = document.createElement('div');
    div.className = 'cell';
    div.dataset.idx = String(idx);
    div.textContent = symbol;
    div.addEventListener('pointerdown', (ev)=>{ ev.preventDefault(); onCellPressed(idx); });
    elementsWrap.appendChild(div);
  });
}

/* ---------- Secuencia ---------- */
function generateSequence(len){
  const poolLen = pools[currentRound - 1].length;
  return Array.from({length:len}, ()=> Math.floor(Math.random()*poolLen));
}
async function showSequence(seq){
  listening = false;
  setBubble('Observa la secuencia...', 2000);
  await new Promise(r=>setTimeout(r, 2000));
  const cells = Array.from(elementsWrap.querySelectorAll('.cell'));
  for(let idx of seq){
    const cell = cells[idx];
    if(cell){
      cell.classList.add('active');
      safePlay(sndSeq);
      await new Promise(r=>setTimeout(r, 700));
      cell.classList.remove('active');
      await new Promise(r=>setTimeout(r, 150));
    }
  }
  listening = true;
  setBubble('¡Tu turno! Repite la secuencia', 2200);
}

/* ---------- Rondas ---------- */
function startRound(){
  if(currentRound > pools.length){ return gameOverWin(); }
  sequenceCount = 0;
  setBubble('Ronda ' + currentRound + ' — 2 secuencias', 2500);
  buildBoard(currentRound);
  setTimeout(()=> startNewSequence(), 1000);
}
function startNewSequence(){
  const len = 2 + (currentRound - 1) + sequenceCount;
  sequence = generateSequence(len);
  playerIndex = 0;
  showSequence(sequence);
}

/* ---------- Jugador ---------- */
function onCellPressed(idx){
  if(!listening) return;
  if(idx !== sequence[playerIndex]){
    safePlay(sndErr);
    lives = Math.max(0, lives - 1);
    updateHUD();
    livesDisplay.classList.add('lifeLost');
    setTimeout(()=> livesDisplay.classList.remove('lifeLost'), 1600);
    setBubble('¡Oh no! Perdimos una vida 🌱', 2500);
    listening = false; playerIndex = 0;
    if(lives <= 0){ return setTimeout(()=> gameOverLose(), 700); }
    else { return setTimeout(()=> showSequence(sequence), 1000); }
  }
  safePlay(sndOk);
  const cell = elementsWrap.querySelectorAll('.cell')[idx];
  if(cell){ cell.classList.add('active'); setTimeout(()=>cell.classList.remove('active'),220); }
  playerIndex++; score += 10; updateHUD();
  if(playerIndex >= sequence.length){
    score += 20; updateHUD();
    sequenceCount++; listening=false;
    if(sequenceCount < 2){
      setBubble('¡Muy bien! Otra secuencia.', 1800);
      setTimeout(()=> startNewSequence(), 1200);
    } else {
      setBubble('¡Ronda completada! 🎉', 1800);
      currentRound++; setTimeout(()=> startRound(), 1500);
    }
  }
}

/* ---------- Game over ---------- */
function gameOverWin(){
  gameContainer.style.display = 'none'; winOverlay.style.display = 'flex';
  setBubble('¡Tu jardín florece! 🌸', 3000); safePlay(sndOk);
}
function gameOverLose(){
  gameContainer.style.display = 'none'; loseOverlay.style.display = 'flex';
  setBubble('¡Ánimo! Intenta otra vez.', 3000); safePlay(sndErr);
}

/* ---------- Botones ---------- */
function startBtnClicked(){
  startScreen.style.display = 'none'; gameContainer.style.display = 'block';
  currentRound=1; score=0; lives=3; sequenceCount=0; playerIndex=0;
  updateHUD(); setTimeout(()=> startRound(), 400);
}
startBtn.addEventListener('click', startBtnClicked);
restartBtn.addEventListener('click', ()=>{ winOverlay.style.display='none'; loseOverlay.style.display='none'; gameContainer.style.display='block'; startBtnClicked(); });
winRestart.addEventListener('click', ()=>{ winOverlay.style.display='none'; startBtnClicked(); });
loseRestart.addEventListener('click', ()=>{ loseOverlay.style.display='none'; startBtnClicked(); });

/* inicial */
updateHUD();
setBubble('¡Hola! Presiona "Comenzar Juego".', 4000);
</script>
</body>
</html>
