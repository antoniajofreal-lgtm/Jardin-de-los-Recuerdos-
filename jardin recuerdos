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
  html,body{height:100%;margin:0;font-family: "Comic Sans MS", system-ui, Arial;}
  body{
    background: url("https://upload.wikimedia.org/wikipedia/commons/6/6e/Field_in_summer.JPG") center/cover no-repeat fixed;
    color:#153e2e;
  }

  .screen { position:fixed; inset:0; display:flex; flex-direction:column; align-items:center; justify-content:center; padding:20px; box-sizing:border-box; }
  .panel { background: rgba(255,255,255,0.7); border-radius:12px; padding:18px; text-align:center; box-shadow:0 8px 30px rgba(0,0,0,0.12); }

  h1 { margin:0 0 12px; font-size:2rem; color:#1f6b34; }

  #gameContainer { display:none; }
  .hud { position: absolute; top:12px; left:12px; right:12px; display:flex; justify-content:space-between; align-items:center; gap:12px; }
  .hud .box { background: rgba(255,255,255,0.85); padding:8px 12px; border-radius:10px; font-weight:700; color:#16522f; }
  .hud button { background:#6ca3f5; color:white; border:0; padding:8px 12px; border-radius:8px; cursor:pointer; }

  #grid { display:grid; grid-template-columns: repeat(4, var(--cell-size)); gap:14px; justify-content:center; margin-top:80px; padding-bottom: var(--bottom-space); }
  .cell { width:var(--cell-size); height:var(--cell-size); background:rgba(255,255,255,0.97); border-radius:14px; display:flex; align-items:center; justify-content:center; box-shadow:0 8px 18px rgba(0,0,0,0.12); cursor:pointer; user-select:none; }
  .cell img { max-width:78%; max-height:78%; pointer-events:none; user-select:none; }
  .cell.active { transform: scale(1.12); background:#dff7e6; transition: all 0.14s ease; }

  #gardenerWrap { position:fixed; right:14px; bottom:12px; display:flex; flex-direction:column; align-items:center; gap:8px; z-index:800; pointer-events:none; }
  #speech { min-width:160px; max-width:240px; background:rgba(255,255,255,0.95); color:#173d2a; padding:10px 12px; border-radius:11px; font-size:15px; box-shadow:0 10px 26px rgba(0,0,0,0.12); text-align:center; }
  #gardenerImg { width:116px; height:auto; display:block; border-radius:8px; }

  #endOverlay { display:none; position:fixed; inset:0; align-items:center; justify-content:center; z-index:900; }
  #endPanel { background: rgba(255,255,255,0.96); padding:22px; border-radius:14px; text-align:center; box-shadow:0 18px 40px rgba(0,0,0,0.18); }

  .btn { padding:10px 16px; border-radius:10px; border:0; background:#57b86a; color:white; cursor:pointer; font-size:16px; }
  .btn.alt { background:#6ca3f5; }

  @media (max-width:520px){
    :root { --cell-size: 84px; --bottom-space: 160px; }
    h1 { font-size:1.6rem; }
    .cell img { max-width:72%; max-height:72%; }
    #speech { font-size:14px; }
  }
</style>
</head>
<body>

<div id="startScreen" class="screen">
  <div class="panel">
    <h1>🌳 El Jardín de los Recuerdos</h1>
    <p>Ayuda al jardinero a cuidar su jardín recordando secuencias.<br><strong>2 secuencias por ronda — 3 vidas</strong>.</p>
    <div style="margin-top:12px;">
      <button id="startBtn" class="btn">🌸 Comenzar Juego</button>
      <button id="musicBtnStart" class="btn alt">🔊 Música</button>
    </div>
  </div>
</div>

<div id="gameContainer" class="screen" style="display:none; align-items:flex-start;">
  <div class="hud">
    <div class="box" id="roundBox">Ronda 1: Sembrar las frutas</div>
    <div class="box" id="scoreBox">Puntos: 0</div>
    <div class="box" id="livesBox">Vidas: 🌸🌸🌸</div>
    <button id="musicBtnGame">🔊 Música</button>
  </div>

  <div id="grid"></div>

  <div id="gardenerWrap">
    <div id="speech">¡Hola! Presiona "Comenzar Juego".</div>
    <img id="gardenerImg" src="https://upload.wikimedia.org/wikipedia/commons/5/51/Gardener_cartoon.png" alt="Jardinero">
  </div>
</div>

<div id="endOverlay">
  <div id="endPanel">
    <h2 id="endTitle">🌟 Resultado</h2>
    <p id="endText"></p>
    <div style="margin-top:12px;">
      <button id="restartBtn" class="btn">🔄 Jugar otra vez</button>
    </div>
  </div>
</div>

<!-- música y sonidos -->
<audio id="bgMusic" loop src="https://files.freemusicarchive.org/storage-freemusicarchive-org/music/ccCommunity/Jahzzar/Traveller/Jahzzar_-_05_-_Siesta.mp3"></audio>
<audio id="sndTap" src="https://actions.google.com/sounds/v1/cartoon/wood_plank_flicks.ogg"></audio>
<audio id="sndGood" src="https://actions.google.com/sounds/v1/cartoon/clang_and_wobble.ogg"></audio>
<audio id="sndBad" src="https://actions.google.com/sounds/v1/cartoon/metal_thud_and_clang.ogg"></audio>

<script>
const ITEMS = [
  { id:0, src:"https://upload.wikimedia.org/wikipedia/commons/1/15/Red_Apple.jpg", name:"Manzana" },
  { id:1, src:"https://upload.wikimedia.org/wikipedia/commons/8/8a/Banana-Single.jpg", name:"Plátano" },
  { id:2, src:"https://upload.wikimedia.org/wikipedia/commons/3/36/Kyoho-grape.jpg", name:"Uvas" },
  { id:3, src:"https://upload.wikimedia.org/wikipedia/commons/4/4b/Carrot_%28PSF%29.png", name:"Zanahoria" },
  { id:4, src:"https://upload.wikimedia.org/wikipedia/commons/2/29/PerfectStrawberry.jpg", name:"Fresa" },
  { id:5, src:"https://upload.wikimedia.org/wikipedia/commons/c/c4/Orange-Fruit-Pieces.jpg", name:"Naranja" },
  { id:6, src:"https://upload.wikimedia.org/wikipedia/commons/0/03/Broccoli_and_cross_section_edit.jpg", name:"Brócoli" }
];

const ROUND_NAMES = [
  "Ronda 1: Sembrar las frutas 🌱🍎",
  "Ronda 2: Cosechar las frutas 🍌🍇",
  "Ronda 3: A juntar la cosecha 🧺"
];

const MAX_ROUNDS = 3;
const SEQUENCES_PER_ROUND = 2;

let currentRound=1, sequence=[], playerIndex=0, sequenceCount=0, score=0, lives=3, listening=false, musicOn=false;

const startScreen=document.getElementById('startScreen');
const startBtn=document.getElementById('startBtn');
const musicBtnStart=document.getElementById('musicBtnStart');
const gameContainer=document.getElementById('gameContainer');
const grid=document.getElementById('grid');
const roundBox=document.getElementById('roundBox');
const scoreBox=document.getElementById('scoreBox');
const livesBox=document.getElementById('livesBox');
const musicBtnGame=document.getElementById('musicBtnGame');
const speech=document.getElementById('speech');
const endOverlay=document.getElementById('endOverlay');
const endTitle=document.getElementById('endTitle');
const endText=document.getElementById('endText');
const restartBtn=document.getElementById('restartBtn');

const bgMusic=document.getElementById('bgMusic');
const sndTap=document.getElementById('sndTap');
const sndGood=document.getElementById('sndGood');
const sndBad=document.getElementById('sndBad');

function safePlay(a){try{a.currentTime=0;a.play().catch(()=>{});}catch(e){}}

function updateHUD(){
  roundBox.textContent=ROUND_NAMES[currentRound-1]||"¡Juego completado!";
  scoreBox.textContent=`Puntos: ${score}`;
  livesBox.textContent=`Vidas: ${'🌸'.repeat(lives)}`;
}

function setSpeech(t,ms=4000){speech.textContent=t;setTimeout(()=>{if(speech.textContent===t)speech.textContent='';},ms);}

function buildGrid(){
  grid.innerHTML='';
  ITEMS.forEach((item,idx)=>{
    const cell=document.createElement('div');
    cell.className='cell';
    cell.dataset.idx=idx;
    const img=document.createElement('img');
    img.src=item.src; img.alt=item.name;
    cell.appendChild(img);
    cell.addEventListener('pointerdown',()=>handlePlayerPress(idx,cell));
    grid.appendChild(cell);
  });
}

function generateSequence(){const len=(currentRound+1)+sequenceCount;return Array.from({length:len},()=>Math.floor(Math.random()*ITEMS.length));}

async function showSequence(seq){
  listening=false;setSpeech('Observa la secuencia...',2200);
  await new Promise(r=>setTimeout(r,2200));
  const cells=[...grid.querySelectorAll('.cell')];
  for(let i=0;i<seq.length;i++){
    const idx=seq[i];const cell=cells[idx];
    cell.classList.add('active');safePlay(sndTap);
    await new Promise(r=>setTimeout(r,650));
    cell.classList.remove('active');
    await new Promise(r=>setTimeout(r,140));
  }
  listening=true;setSpeech('¡Tu turno! Repite la secuencia',2200);
  playerIndex=0;
}

function startRound(){if(currentRound>MAX_ROUNDS)return finishGame(true);
  sequenceCount=0;updateHUD();setSpeech(ROUND_NAMES[currentRound-1],4500);buildGrid();
  setTimeout(()=>startNewSequence(),900);
}
function startNewSequence(){sequence=generateSequence();playerIndex=0;showSequence(sequence);}
function handlePlayerPress(idx,cell){
  if(!listening)return;safePlay(sndTap);
  cell.classList.add('active');setTimeout(()=>cell.classList.remove('active'),220);
  if(idx!==sequence[playerIndex]){
    safePlay(sndBad);lives--;updateHUD();setSpeech('¡Oh! Esa no es la correcta.',2600);
    listening=false;playerIndex=0;if(lives<=0)return setTimeout(()=>finishGame(false),700);
    else return setTimeout(()=>showSequence(sequence),900);
  }
  playerIndex++;score+=10;updateHUD();
  if(playerIndex>=sequence.length){
    safePlay(sndGood);score+=20;updateHUD();listening=false;sequenceCount++;
    if(sequenceCount<SEQUENCES_PER_ROUND){setSpeech('¡Muy bien! Ahora otra secuencia.',3800);setTimeout(()=>startNewSequence(),1100);}
    else {setSpeech('¡Ronda completada! 🎉',2200);currentRound++;setTimeout(()=>{if(currentRound>MAX_ROUNDS)finishGame(true);else startRound();},1200);}
  }
}
function finishGame(won){
  gameContainer.style.display='none';endOverlay.style.display='flex';
  if(won){endTitle.textContent="🌟 ¡Lo lograste! 🌟";endText.textContent=`Completaste el jardín. Puntaje final: ${score}`;}
  else {endTitle.textContent="💪 ¡Ánimo! 💪";endText.textContent=`Perdiste todas las vidas. Puntaje final: ${score}`;}
}
async function toggleMusic(){if(musicOn){bgMusic.pause();musicOn=false;}else{await safePlay(bgMusic);musicOn=!bgMusic.paused;}
musicBtnStart.textContent=musicOn?'🔇 Música':'🔊 Música';musicBtnGame.textContent=musicOn?'🔇 Música':'🔊 Música';}
startBtn.addEventListener('click',()=>{startScreen.style.display='none';gameContainer.style.display='block';endOverlay.style.display='none';
currentRound=1;sequence=[];playerIndex=0;sequenceCount=0;score=0;lives=3;listening=false;updateHUD();buildGrid();safePlay(bgMusic);musicOn=true;
musicBtnStart.textContent='🔇 Música';musicBtnGame.textContent='🔇 Música';setTimeout(()=>startRound(),450);});
musicBtnStart.addEventListener('click',toggleMusic);musicBtnGame.addEventListener('click',toggleMusic);
restartBtn.addEventListener('click',()=>{endOverlay.style.display='none';startScreen.style.display='flex';gameContainer.style.display='none';setSpeech('¡Hola! Presiona "Comenzar Juego".');});
updateHUD();buildGrid();setSpeech('¡Hola! Presiona "Comenzar Juego".',4000);
</script>
</body>
</html>
