<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>El Jardín de los Recuerdos</title>
<style>
  body {
    margin:0;
    font-family: 'Comic Sans MS', cursive, sans-serif;
    background: url("https://cdn.pixabay.com/photo/2016/11/29/09/32/field-1866864_1280.jpg") no-repeat center center fixed;
    background-size: cover;
    overflow:hidden;
  }
  #startScreen, #gameContainer, #endScreen {
    position: absolute; top:0; left:0; width:100%; height:100%;
    display:flex; flex-direction:column; align-items:center; justify-content:center;
    background: rgba(255,255,255,0.3);
    text-align:center;
  }
  #startScreen h1, #endScreen h1 {
    font-size: 2.5em;
    color: darkgreen;
    margin-bottom:20px;
    text-shadow:1px 1px 3px white;
  }
  button {
    padding:10px 20px;
    margin:10px;
    font-size:1.2em;
    border:none;
    border-radius:10px;
    cursor:pointer;
    background:green; color:white;
    box-shadow:2px 2px 5px rgba(0,0,0,0.3);
  }
  #hud {
    position:absolute; top:10px; left:10px; right:10px;
    display:flex; justify-content:space-between; align-items:center;
    font-size:1.2em; color:darkgreen; font-weight:bold;
    text-shadow:1px 1px 2px white;
  }
  #lives span {
    font-size:1.5em;
  }
  #grid {
    display:grid;
    grid-template-columns: repeat(4,100px);
    grid-gap:15px;
    margin-top:100px;
  }
  .cell {
    width:100px; height:100px;
    display:flex; align-items:center; justify-content:center;
    background:white;
    border-radius:15px;
    box-shadow:2px 2px 5px rgba(0,0,0,0.3);
    cursor:pointer;
    transition:transform 0.2s, background 0.2s;
  }
  .cell img {
    max-width:80px;
    max-height:80px;
    user-select:none;
    pointer-events:none;
  }
  .highlight {
    background:yellow;
    transform:scale(1.15);
  }
  #gardener {
    position:absolute; bottom:20px; left:20px;
    width:160px;
    text-align:center;
  }
  #gardener img {
    width:100%;
    max-width:160px;
  }
  #speech {
    background:white; padding:10px;
    border-radius:10px;
    margin-top:5px;
    font-size:1em;
    box-shadow:2px 2px 5px rgba(0,0,0,0.3);
  }
  #endScreen {
    display:none;
    text-align:center;
  }
  #finalScore {
    font-size:1.5em;
    margin:15px;
    color:darkblue;
  }
</style>
</head>
<body>

<div id="startScreen">
  <h1>🌸 El Jardín de los Recuerdos 🌸</h1>
  <button id="startBtn">Comenzar Juego</button>
  <button id="musicBtnStart">🔊 Música</button>
</div>

<div id="gameContainer" style="display:none;">
  <div id="hud">
    <div id="round">Ronda: 1</div>
    <div id="score">Puntos: 0</div>
    <div id="lives">🌼🌼🌼</div>
    <button id="musicBtnGame">🔊 Música</button>
  </div>
  <div id="grid"></div>
  <div id="gardener">
    <img src="https://cdn.pixabay.com/photo/2013/07/12/18/39/farmer-153020_1280.png" alt="jardinero">
    <div id="speech">¡Bienvenido al jardín!</div>
  </div>
</div>

<div id="endScreen">
  <h1>🌟 Fin del Juego 🌟</h1>
  <div id="finalScore"></div>
  <button id="restartBtn">🔄 Volver a Jugar</button>
</div>

<audio id="bgMusic" loop>
  <source src="https://cdn.pixabay.com/download/audio/2022/03/15/audio_6f58b5469d.mp3?filename=happy-background-110111.mp3" type="audio/mpeg">
</audio>

<audio id="sound1" src="https://cdn.pixabay.com/download/audio/2022/03/15/audio_0f2c22fc52.mp3?filename=click-110123.mp3"></audio>
<audio id="sound2" src="https://cdn.pixabay.com/download/audio/2021/09/01/audio_7f1b5c9f1b.mp3?filename=water-drop-1-109434.mp3"></audio>
<audio id="sound3" src="https://cdn.pixabay.com/download/audio/2022/03/15/audio_2a983a58bd.mp3?filename=pop-110126.mp3"></audio>

<script>
const grid = document.getElementById("grid");
const roundDisplay = document.getElementById("round");
const scoreDisplay = document.getElementById("score");
const livesDisplay = document.getElementById("lives");
const speech = document.getElementById("speech");
const startScreen = document.getElementById("startScreen");
const gameContainer = document.getElementById("gameContainer");
const endScreen = document.getElementById("endScreen");
const finalScore = document.getElementById("finalScore");
const startBtn = document.getElementById("startBtn");
const restartBtn = document.getElementById("restartBtn");
const musicBtnStart = document.getElementById("musicBtnStart");
const musicBtnGame = document.getElementById("musicBtnGame");
const bgMusic = document.getElementById("bgMusic");
const sounds = [document.getElementById("sound1"), document.getElementById("sound2"), document.getElementById("sound3")];

const items = [
  "https://cdn.pixabay.com/photo/2016/08/05/17/33/apple-1571994_1280.png",
  "https://cdn.pixabay.com/photo/2014/04/02/10/56/banana-303688_1280.png",
  "https://cdn.pixabay.com/photo/2017/01/23/19/25/grapes-2002938_1280.png",
  "https://cdn.pixabay.com/photo/2012/04/12/23/47/carrot-30994_1280.png",
  "https://cdn.pixabay.com/photo/2014/04/02/16/18/strawberry-307831_1280.png",
  "https://cdn.pixabay.com/photo/2016/08/05/17/32/orange-1571993_1280.png",
  "https://cdn.pixabay.com/photo/2016/03/05/22/34/broccoli-1238250_1280.png"
];

let sequence = [];
let playerSequence = [];
let currentRound = 1;
let score = 0;
let lives = 3;
let acceptingInput = false;

// Crear grid
function createGrid(){
  grid.innerHTML="";
  items.forEach((src, idx)=>{
    const cell = document.createElement("div");
    cell.classList.add("cell");
    cell.dataset.index=idx;
    const img = document.createElement("img");
    img.src = src;
    cell.appendChild(img);
    cell.addEventListener("click", ()=>handlePlayerInput(idx));
    grid.appendChild(cell);
  });
}

// Mostrar secuencia
async function showSequence(){
  acceptingInput=false;
  for(let i=0;i<sequence.length;i++){
    const idx=sequence[i];
    const cell=grid.querySelector(`[data-index='${idx}']`);
    await new Promise(res=>{
      setTimeout(()=>{
        cell.classList.add("highlight");
        sounds[i % sounds.length].play();
        setTimeout(()=>{
          cell.classList.remove("highlight");
          res();
        },600);
      },600);
    });
  }
  acceptingInput=true;
}

// Manejar jugada
function handlePlayerInput(idx){
  if(!acceptingInput) return;
  const sound = sounds[Math.floor(Math.random()*sounds.length)];
  sound.play();

  playerSequence.push(idx);
  const currentStep = playerSequence.length-1;

  if(playerSequence[currentStep] !== sequence[currentStep]){
    lives--;
    updateHUD();
    playerSequence=[];
    if(lives<=0) return endGame();
    showMessage("¡Ups! Pierdes una flor 🌼");
    return;
  }

  if(playerSequence.length === sequence.length){
    score += sequence.length*10;
    updateHUD();
    playerSequence=[];
    if(sequence.length>=3+currentRound){ // longitud meta
      currentRound++;
      if(currentRound>3) return endGame();
      showMessage("¡Bien! Vamos a la siguiente ronda...");
      setTimeout(startRound,2500);
    } else {
      nextTurn();
    }
  }
}

// HUD
function updateHUD(){
  roundDisplay.textContent=`Ronda: ${currentRound}`;
  scoreDisplay.textContent=`Puntos: ${score}`;
  livesDisplay.textContent="🌼".repeat(lives);
}

// Nueva ronda
function startRound(){
  sequence=[];
  playerSequence=[];
  if(currentRound===1) showMessage("🌱 Ronda 1: ¡A sembrar las frutas!");
  if(currentRound===2) showMessage("🍊 Ronda 2: ¡A cosechar las frutas!");
  if(currentRound===3) showMessage("🥕 Ronda 3: ¡A juntar la cosecha!");
  nextTurn();
}

// Añadir un paso más
function nextTurn(){
  sequence.push(Math.floor(Math.random()*items.length));
  showSequence();
}

// Final del juego
function endGame(){
  gameContainer.style.display="none";
  endScreen.style.display="flex";
  finalScore.textContent=`Tu puntaje final: ${score}`;
}

// Mensajes jardinero
function showMessage(msg){
  speech.textContent=msg;
}

// Música
let musicOn=false;
async function toggleMusic(){
  if(musicOn){
    bgMusic.pause();
    musicOn=false;
  } else {
    try {
      await bgMusic.play();
      musicOn=true;
    } catch(e){
      console.log("El navegador bloqueó la música hasta interacción directa.");
    }
  }
  musicBtnStart.textContent=musicOn?'🔇 Música':'🔊 Música';
  musicBtnGame.textContent=musicOn?'🔇 Música':'🔊 Música';
}

// Eventos
startBtn.addEventListener("click", async ()=>{
  startScreen.style.display="none";
  gameContainer.style.display="block";
  currentRound=1; score=0; lives=3; updateHUD();
  createGrid();
  try { await bgMusic.play(); musicOn=true; } catch(e){}
  setTimeout(startRound,500);
});
restartBtn.addEventListener("click", ()=>{
  endScreen.style.display="none";
  startScreen.style.display="flex";
});
musicBtnStart.addEventListener("click", toggleMusic);
musicBtnGame.addEventListener("click", toggleMusic);
</script>
</body>
</html>
