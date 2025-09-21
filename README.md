<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="utf-8"/>
<meta name="viewport" content="width=device-width,initial-scale=1"/>
<title>El Jardín de los Recuerdos</title>
<style>
  :root {
    --cell-size: 100px;
    --bottom-space: 200px;
  }
  /* Fondo y tipografía */
  html,body{height:100%;margin:0;font-family: "Comic Sans MS", system-ui, -apple-system, "Segoe UI", Roboto, Arial;}
  body{
    background: url("https://cdn.pixabay.com/photo/2016/11/29/09/32/field-1866864_1280.jpg") center/cover no-repeat fixed;
    color:#153e2e;
    -webkit-font-smoothing:antialiased;
    -moz-osx-font-smoothing:grayscale;
  }

  /* Contenedores */
  .screen {
    position:fixed; inset:0; display:flex; flex-direction:column; align-items:center; justify-content:center;
    padding:20px; box-sizing:border-box;
  }
  /* semi-transparente ligero para que el fondo se vea */
  .panel { background: rgba(255,255,255,0.32); border-radius:12px; padding:18px; text-align:center; box-shadow:0 8px 30px rgba(0,0,0,0.12); }

  h1 { margin:0 0 12px; font-size:2rem; color:#1f6b34; text-shadow: 1px 1px 2px rgba(255,255,255,0.6); }

  /* HUD */
  #gameContainer { display:none; }
  .hud { position: absolute; top:12px; left:12px; right:12px; display:flex; justify-content:space-between; align-items:center; gap:12px; }
  .hud .box { background: rgba(255,255,255,0.85); padding:8px 12px; border-radius:10px; font-weight:700; color:#16522f; box-shadow:0 6px 14px rgba(0,0,0,0.08); }
  .hud button { background:#6ca3f5; color:white; border:0; padding:8px 12px; border-radius:8px; cursor:pointer; }

  /* Grid */
  #grid { display:grid; grid-template-columns: repeat(4, var(--cell-size)); gap:14px; justify-content:center; margin-top:80px; padding-bottom: var(--bottom-space); }
  .cell { width:var(--cell-size); height:var(--cell-size); background:rgba(255,255,255,0.97); border-radius:14px; display:flex; align-items:center; justify-content:center; box-shadow:0 8px 18px rgba(0,0,0,0.12); cursor:pointer; touch-action: manipulation; user-select:none; }
  .cell img { max-width:78%; max-height:78%; pointer-events:none; user-select:none; }
  .cell.active { transform: scale(1.12); background:#dff7e6; transition: all 0.14s ease; }

  /* Jardinero */
  #gardenerWrap { position:fixed; right:14px; bottom:12px; display:flex; flex-direction:column; align-items:center; gap:8px; z-index:800; pointer-events:none; }
  #speech { min-width:160px; max-width:240px; background:rgba(255,255,255,0.98); color:#173d2a; padding:10px 12px; border-radius:11px; font-size:15px; box-shadow:0 10px 26px rgba(0,0,0,0.12); text-align:center; }
  #gardenerImg { width:116px; height:auto; display:block; border-radius:8px; }

  /* overlay final */
  #endOverlay { display:none; position:fixed; inset:0; align-items:center; justify-content:center; z-index:900; }
  #endPanel { background: rgba(255,255,255,0.96); padding:22px; border-radius:14px; text-align:center; box-shadow:0 18px 40px rgba(0,0,0,0.18); }

  /* botones */
  .btn { padding:10px 16px; border-radius:10px; border:0; background:#57b86a; color:white; cursor:pointer; font-size:16px; }
  .btn.alt { background:#6ca3f5; }

  /* responsive */
  @media (max-width:520px){
    :root { --cell-size: 84px; --bottom-space: 160px; }
    h1 { font-size:1.6rem; }
    .cell img { max-width:72%; max-height:72%; }
    #speech { font-size:14px; }
  }
</style>
</head>
<body>

  <!-- pantalla de inicio -->
  <div id="startScreen" class="screen">
    <div class="panel">
      <h1>🌳 El Jardín de los Recuerdos</h1>
      <p>Ayuda al jardinero a cuidar su jardín recordando secuencias. <br><strong>2 secuencias por ronda — 3 vidas</strong>.</p>
      <div style="margin-top:12px;">
        <button id="startBtn" class="btn">🌸 Comenzar Juego</button>
        <button id="musicBtnStart" class="btn alt">🔊 Música</button>
      </div>
    </div>
  </div>

  <!-- pantalla de juego -->
  <div id="gameContainer" class="screen" style="display:none; align-items:flex-start;">
    <div class="hud" aria-hidden="false" role="region">
      <div class="box" id="roundBox">Ronda 1: Sembrar las frutas</div>
      <div class="box" id="scoreBox">Puntos: 0</div>
      <div class="box" id="livesBox">Vidas: 🌸🌸🌸</div>
      <button id="musicBtnGame" class="">🔊 Música</button>
    </div>

    <div id="grid" role="application" aria-label="Tablero de elementos"></div>

    <!-- jardinero y globo -->
    <div id="gardenerWrap" aria-hidden="true">
      <div id="speech" style="display:block">¡Hola! Presiona "Comenzar Juego".</div>
      <img id="gardenerImg" src="https://cdn.pixabay.com/photo/2014/04/03/11/53/gardener-311325_1280.png" alt="Jardinero">
    </div>
  </div>

  <!-- overlay final -->
  <div id="endOverlay">
    <div id="endPanel">
      <h2 id="endTitle">🌟 Resultado</h2>
      <p id="endText"></p>
      <div style="margin-top:12px;">
        <button id="restartBtn" class="btn">🔄 Jugar otra vez</button>
      </div>
    </div>
  </div>

  <!-- audios -->
  <audio id="bgMusic" loop src="https://files.freemusicarchive.org/storage-freemusicarchive-org/music/ccCommunity/Jahzzar/Traveller/Jahzzar_-_05_-_Siesta.mp3"></audio>
  <!-- sonido para mostrar y para pulsar correcto (mismo) -->
  <audio id="sndTap" src="https://actions.google.com/sounds/v1/cartoon/wood_plank_flicks.ogg"></audio>
  <!-- acierto de secuencia completa -->
  <audio id="sndGood" src="https://actions.google.com/sounds/v1/cartoon/clang_and_wobble.ogg"></audio>
  <!-- error -->
  <audio id="sndBad" src="https://actions.google.com/sounds/v1/cartoon/metal_thud_and_clang.ogg"></audio>

<script>
/* -------------------------------
   Configuración / recursos
   ------------------------------- */

/* Imágenes estables (Pixabay) - 7 items */
const ITEMS = [
  { id:0, src:"https://cdn.pixabay.com/photo/2016/08/05/17/33/apple-1571994_1280.png", name:"Manzana" },
  { id:1, src:"https://cdn.pixabay.com/photo/2014/04/02/10/56/banana-303688_1280.png", name:"Plátano" },
  { id:2, src:"https://cdn.pixabay.com/photo/2017/01/23/19/25/grapes-2002938_1280.png", name:"Uvas" },
  { id:3, src:"https://cdn.pixabay.com/photo/2012/04/12/23/47/carrot-30994_1280.png", name:"Zanahoria" },
  { id:4, src:"https://cdn.pixabay.com/photo/2014/04/02/16/18/strawberry-307831_1280.png", name:"Fresa" },
  { id:5, src:"https://cdn.pixabay.com/photo/2016/08/05/17/32/orange-1571993_1280.png", name:"Naranja" },
  { id:6, src:"https://cdn.pixabay.com/photo/2016/03/05/22/34/broccoli-1238250_1280.png", name:"Brócoli" }
];

/* Rondas - nombres solicitados */
const ROUND_NAMES = [
  "Ronda 1: Sembrar las frutas",
  "Ronda 2: Cosechar las frutas",
  "Ronda 3: A juntar la cosecha"
];

const MAX_ROUNDS = 3;
const SEQUENCES_PER_ROUND = 2;

/* Elementos DOM */
const startScreen = document.getElementById('startScreen');
const startBtn = document.getElementById('startBtn');
const musicBtnStart = document.getElementById('musicBtnStart');

const gameContainer = document.getElementById('gameContainer');
const grid = document.getElementById('grid');
const roundBox = document.getElementById('roundBox');
const scoreBox = document.getElementById('scoreBox');
const livesBox = document.getElementById('livesBox');
const musicBtnGame = document.getElementById('musicBtnGame');

const speech = document.getElementById('speech');
const gardenerImg = document.getElementById('gardenerImg');

const endOverlay = document.getElementById('endOverlay');
const endTitle = document.getElementById('endTitle');
const endText = document.getElementById('endText');
const restartBtn = document.getElementById('restartBtn');

/* Audios */
const bgMusic = document.getElementById('bgMusic');
const sndTap = document.getElementById('sndTap');
const sndGood = document.getElementById('sndGood');
const sndBad = document.getElementById('sndBad');

/* Estado del juego */
let currentRound = 1;
let sequence = [];        // array de índices (0..6)
let playerIndex = 0;      // posición actual que el jugador debe acertar
let sequenceCount = 0;    // cuántas secuencias ya completadas en la ronda (0..SEQUENCES_PER_ROUND-1)
let score = 0;
let lives = 3;
let listening = false;    // si el juego acepta toques del jugador
let musicOn = false;

/* -------------------------------
   Utilidades
   ------------------------------- */
function emojiFallbackSVG(emoji='🧑‍🌾',size=160){
  const svg = `<svg xmlns='http://www.w3.org/2000/svg' width='${size}' height='${size}'><rect width='100%' height='100%' fill='%23ffffff'/><text x='50%' y='50%' dominant-baseline='middle' text-anchor='middle' font-size='72'>${emoji}</text></svg>`;
  return 'data:image/svg+xml;utf8,' + encodeURIComponent(svg);
}

function safePlay(audioEl){
  if(!audioEl) return Promise.resolve();
  try{
    audioEl.currentTime = 0;
    return audioEl.play().catch(e=>{
      // bloqueado por política de autoplay - ignoramos
      console.debug('audio play blocked', e);
    });
  }catch(e){
    return Promise.resolve();
  }
}

/* -------------------------------
   UI / HUD
   ------------------------------- */
function updateHUD(){
  roundBox.textContent = (currentRound <= MAX_ROUNDS) ? ROUND_NAMES[currentRound-1] : "¡Juego completado!";
  scoreBox.textContent = `Puntos: ${score}`;
  livesBox.textContent = `Vidas: ${'🌸'.repeat(Math.max(0,lives))}`;
}

function setSpeech(text, ms=4000){
  speech.textContent = text;
  // mostramos por ms si no se reemplaza
  setTimeout(()=>{ if(speech.textContent === text) speech.textContent = ''; }, ms);
}

/* -------------------------------
   Grid / items
   ------------------------------- */
function buildGrid(){
  grid.innerHTML = '';
  ITEMS.forEach((item, idx) => {
    const cell = document.createElement('div');
    cell.className = 'cell';
    cell.dataset.idx = String(idx);
    const img = document.createElement('img');
    img.src = item.src;
    img.alt = item.name;
    // fallback si imagen falla
    img.addEventListener('error', ()=>{ img.src = emojiFallbackSVG('🍎', 120); });
    cell.appendChild(img);

    // NUEVO: pointerdown para reducir latencia táctil; si no disponible, se usa click
    cell.addEventListener('pointerdown', (ev)=>{
      ev.preventDefault();
      handlePlayerPress(idx, cell);
    });
    grid.appendChild(cell);
  });
}

/* -------------------------------
   Secuencia (2 secuencias por ronda)
   ------------------------------- */
function generateSequenceForRound(){
  // longitud base: round + 1 for first sequence, + sequenceCount for second (grows by 1)
  const len = (currentRound + 1) + sequenceCount; // same logic as prior working versions
  const poolLen = ITEMS.length;
  const seq = [];
  for(let i=0;i<len;i++){
    seq.push(Math.floor(Math.random() * poolLen));
  }
  return seq;
}

async function showSequence(seq){
  listening = false;
  setSpeech('Observa la secuencia...', 2200);
  // small wait for speech
  await new Promise(r => setTimeout(r, 2200));

  const cells = Array.from(grid.querySelectorAll('.cell'));
  for(let i=0;i<seq.length;i++){
    const idx = seq[i];
    const cell = cells[idx];
    if(cell){
      cell.classList.add('active');
      await safePlay(sndTap);           // mismo sonido al mostrar
      await new Promise(r => setTimeout(r, 650));
      cell.classList.remove('active');
      await new Promise(r => setTimeout(r, 140));
    } else {
      // seguridad: si algo raro pasa
      await new Promise(r => setTimeout(r, 700));
    }
  }
  listening = true;
  setSpeech('¡Tu turno! Repite la secuencia', 2200);
  playerIndex = 0;
}

/* -------------------------------
   Flujo de rondas / secuencias
   ------------------------------- */
function startRound(){
  if(currentRound > MAX_ROUNDS){
    // gana
    finishGame(true);
    return;
  }
  sequenceCount = 0; // reinicia las secuencias por ronda
  updateHUD();
  setSpeech(ROUND_NAMES_TEXT(currentRound), 4500);
  // construimos grid (estático, pero rebuild seguro)
  buildGrid();
  // arrancamos primera secuencia dentro de la ronda
  setTimeout(()=> startNewSequence(), 900);
}

function START_NEW_SEQUENCE_INTERNAL(){
  sequence = generateSequenceForRound();
  playerIndex = 0;
  showSequence(sequence).catch(e=>console.debug(e));
}

function startNewSequence(){
  START_NEW_SEQUENCE_INTERNAL();
}

/* -------------------------------
   Manejo de interacción del jugador
   ------------------------------- */
function handlePlayerPress(idx, cellEl){
  if(!listening) return;
  // comparar índice directamente (evita falsos negativos)
  // reproducir mismo sonido que la secuencia
  safePlay(sndTap);
  // efecto visual inmediato
  cellEl.classList.add('active');
  setTimeout(()=> cellEl.classList.remove('active'), 220);

  if(idx !== sequence[playerIndex]){
    // error
    safePlay(sndBad);
    lives = Math.max(0, lives - 1);
    updateHUD();
    setSpeech('¡Oh! Esa no es la correcta.', 2600);
    listening = false;
    playerIndex = 0;
    if(lives <= 0){
      setTimeout(()=> finishGame(false), 700);
    } else {
      // repetir la misma secuencia
      setTimeout(()=> showSequence(sequence), 900);
    }
    return;
  }

  // correcto en este paso
  playerIndex++;
  score += 10; // puntos por cada acierto de paso
  updateHUD();

  if(playerIndex >= sequence.length){
    // completó la secuencia completa
    safePlay(sndGood);
    score += 20; // bonus por completar la secuencia
    updateHUD();
    listening = false;
    sequenceCount++;
    if(sequenceCount < SEQUENCES_PER_ROUND){
      // otra secuencia en la misma ronda
      setSpeech('¡Muy bien! Ahora otra secuencia.', 3800);
      setTimeout(()=> startNewSequence(), 1100);
    } else {
      // ronda completada -> avanzar
      setSpeech('¡Ronda completada! 🎉', 2200);
      currentRound++;
      setTimeout(()=> {
        if(currentRound > MAX_ROUNDS) finishGame(true);
        else startRound();
      }, 1200);
    }
  }
}

/* -------------------------------
   Final del juego
   ------------------------------- */
function finishGame(won){
  listening = false;
  // ocultar juego y mostrar overlay
  gameContainer.style.display = 'none';
  endOverlay.style.display = 'flex';
  endOverlay.style.alignItems = 'center';
  endOverlay.style.justifyContent = 'center';
  if(won){
    endTitle.textContent = "🌟 ¡Lo lograste! 🌟";
    endText.textContent = `Completaste el jardín. Puntaje final: ${score}`;
  } else {
    endTitle.textContent = "💪 ¡Ánimo! 💪";
    endText.textContent = `Perdiste todas las vidas. Puntaje final: ${score}`;
  }
}

/* -------------------------------
   Rondas: textos largos (nombres)
   ------------------------------- */
function ROUND_NAMES_TEXT(roundNumber){
  if(roundNumber === 1) return "Ronda 1: Sembrar las frutas 🌱🍎";
  if(roundNumber === 2) return "Ronda 2: Cosechar las frutas 🍌🍇";
  if(roundNumber === 3) return "Ronda 3: A juntar la cosecha 🧺";
  return "";
}

/* -------------------------------
   Manejo botones / música / reinicio
   ------------------------------- */
async function toggleMusic(){
  if(musicOn){
    try{ bgMusic.pause(); }catch(e){/*ignore*/ }
    musicOn = false;
  } else {
    await safePlay(bgMusic);
    musicOn = !bgMusic.paused;
  }
  // actualizar labels
  musicBtnStart.textContent = musicOn ? '🔇 Música' : '🔊 Música';
  musicBtnGame.textContent = musicOn ? '🔇 Música' : '🔊 Música';
}

/* Fallback de imagen para jardinero */
gardenerImg.addEventListener('error', ()=>{ gardenerImg.src = emojiFallbackSVG('🧑‍🌾', 160); });

/* Eventos botones */
startBtn.addEventListener('pointerdown', (e)=>{ e.preventDefault(); startBtnClick(); });
startBtn.addEventListener('click', startBtnClick);

async function startBtnClick(){
  // ocultar inicio, mostrar juego y reiniciar estado
  startScreen.style.display = 'none';
  endOverlay.style.display = 'none';
  gameContainer.style.display = 'block';
  currentRound = 1; sequence = []; playerIndex = 0; sequenceCount = 0; score = 0; lives = 3;
  listening = false;
  updateHUD();
  buildGrid();
  // intentamos iniciar música (si el navegador lo permite)
  try{ await safePlay(bgMusic); musicOn = !bgMusic.paused; }catch(e){}
  musicBtnStart.textContent = musicOn ? '🔇 Música' : '🔊 Música';
  musicBtnGame.textContent = musicOn ? '🔇 Música' : '🔊 Música';
  setTimeout(()=> startRound(), 450);
}

musicBtnStart.addEventListener('click', toggleMusic);
musicBtnGame.addEventListener('click', toggleMusic);

restartBtn.addEventListener('click', ()=>{
  endOverlay.style.display = 'none';
  startScreen.style.display = 'flex';
  gameContainer.style.display = 'none';
  setSpeech('¡Hola! Presiona "Comenzar Juego".');
});

/* Inicialización */
updateHUD();
buildGrid();
setSpeech('¡Hola! Presiona "Comenzar Juego".', 4000);

console.debug('Juego cargado — versión revisada y corregida.');
</script>
</body>
</html>
