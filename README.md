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
    background: rgba(255,255,255,0.6);
  }
  #startScreen h1, #endScreen h1 {
    font-size: 2.5em;
    color: darkgreen;
    margin-bottom:20px;
  }
  button {
    padding:10px 20px;
    margin:10px;
    font-size:1.2em;
    border:none;
    border-radius:10px;
    cursor:pointer;
    background:green; color:white;
  }
  #hud {
    position:absolute; top:10px; left:10px; right:10px;
    display:flex; justify-content:space-between; align-items:center;
    font-size:1.2em; color:darkgreen; font-weight:bold;
  }
  #lives span {
    font-size:1.5em;
  }
  #grid {
    display:grid;
    grid-template-columns: repeat(4,80px);
    grid-gap:15px;
    margin-top:100px;
  }
  .cell {
    width:80px; height:80px;
    display:flex; align-items:center; justify-content:center;
    font-size:2em;
    background:white;
    border-radius:15px;
    box-shadow:2px 2px 5px rgba(0,0,0,0.3);
    cursor:pointer;
    transition:transform 0.2s, background 0.2s;
  }
  .highlight {
    background:yellow;
    transform:scale(1.1);
  }
  #gardener {
    position:absolute; bottom:20px; left:20px;
    width:150px;
    text-align:center;
  }
  #gardener img {
    width:100%;
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
const icons=["🌸","🌻","🍎","🍐","🍊","🍇","🥕","🍓"];
let sequence=[]; 
let playerSequence=[];
let roundNames = ["Sembrar las flores", "Cosechar las frutas", "Juntar la cosecha"];
let currentRound=1; 
let score=0; 
let lives=3;
let acceptingInput=false;
let sequenceCount=0; 
let musicOn=false;

const startScreen=document.getElementById('startScreen');
const startBtn=document.getElementById('startBtn');
const musicBtnStart=document.getElementById('musicBtnStart');
const gameContainer=document.getElementById('gameContainer');
const grid=document.getElementById('grid');
const roundDisp=document.getElementById('round');
const scoreDisp=document.getElementById('score');
const livesDisp=document.getElementById('lives');
const gardenerSpeech=document.getElementById('speech');
const bgMusic=document.getElementById('bgMusic');
const musicBtnGame=document.getElementById('musicBtnGame');
const endScreen=document.getElementById('endScreen');
const finalScore=document.getElementById('finalScore');
const restartBtn=document.getElementById('restartBtn');
const sounds=[
  document.getElementById('sound1'),
  document.getElementById('sound2'),
  document.getElementById('sound3')
];

function updateHUD(){
  roundDisp.textContent="Ronda "+currentRound+": "+roundNames[currentRound-1];
  scoreDisp.textContent="Puntos: "+score;
  livesDisp.textContent="🌼".repeat(lives);
}

function showSpeech(text, duration=3000){
  gardenerSpeech.textContent=text;
  setTimeout(()=>{gardenerSpeech.textContent="";},duration);
}

function renderElements(){
  grid.innerHTML="";
  icons.forEach((icon,i)=>{
    const div=document.createElement("div");
    div.className="cell";
    div.textContent=icon;
    div.dataset.index=i;
    div.addEventListener("click",()=>handleClick(parseInt(div.dataset.index),div));
    grid.appendChild(div);
  });
}

function playSound(idx){
  sounds[idx % sounds.length].currentTime=0;
  sounds[idx % sounds.length].play();
}

function generateSequence(len){
  let seq=[];
  for(let i=0;i<len;i++){
    seq.push(Math.floor(Math.random()*icons.length));
  }
  return seq;
}

async function playSequence(seq){
  acceptingInput=false;
  for(let i=0;i<seq.length;i++){
    let cell=grid.querySelector(`[data-index='${seq[i]}']`);
    cell.classList.add("highlight");
    playSound(seq[i]);
    await new Promise(r=>setTimeout(r,800));
    cell.classList.remove("highlight");
    await new Promise(r=>setTimeout(r,200));
  }
  acceptingInput=true;
  playerSequence=[];
}

function handleClick(i,div){
  if(!acceptingInput)return;
  playSound(i);
  playerSequence.push(i);
  let currentStep=playerSequence.length-1;
  if(playerSequence[currentStep]!==sequence[currentStep]){
    lives--;
    updateHUD();
    showSpeech("¡Ups! Perdiste una vida.",2000);
    acceptingInput=false;
    if(lives<=0){endGame("¡Juego terminado!");return;}
    else{setTimeout(()=>playSequence(sequence),1500);}
    return;
  }
  if(playerSequence.length===sequence.length){
    score+=10;
    updateHUD();
    sequenceCount++;
    acceptingInput=false;
    if(sequenceCount<2){
      showSpeech("¡Muy bien! Aquí va otra secuencia.",3000);
      setTimeout(startRound,3200);
    } else {
      currentRound++;
      sequenceCount=0;
      if(currentRound>3){
        endGame("¡Felicidades, completaste el juego!");
      } else {
        showSpeech("¡Avanzamos a la siguiente ronda!",4000);
        setTimeout(startRound,4200);
      }
    }
  }
}

function startRound(){
  updateHUD();
  renderElements();
  let len=currentRound+2;
  sequence=generateSequence(len);
  setTimeout(()=>playSequence(sequence),1500);
  showSpeech("Ronda "+currentRound+": "+roundNames[currentRound-1],4000);
}

function endGame(message){
  gameContainer.style.display="none";
  endScreen.style.display="flex";
  finalScore.textContent=message+" Puntaje final: "+score;
}

// Música
async function toggleMusic(){
  if(musicOn){
    bgMusic.pause();
    musicOn=false;
  } else {
    try {
      await bgMusic.play();
      musicOn=true;
    } catch(e) {
      console.log("El navegador bloqueó el audio hasta interacción directa.");
      musicOn=false;
    }
  }
  musicBtnStart.textContent=musicOn?'🔇 Música':'🔊 Música';
  musicBtnGame.textContent=musicOn?'🔇 Música':'🔊 Música';
}

musicBtnStart.onclick=toggleMusic;
musicBtnGame.onclick=toggleMusic;

startBtn.addEventListener('click', async ()=>{
  startScreen.style.display='none';
  gameContainer.style.display='block';
  currentRound=1;score=0;lives=3;sequenceCount=0;updateHUD();
  try {
    await bgMusic.play();
    musicOn=true;
    musicBtnStart.textContent='🔇 Música';
    musicBtnGame.textContent='🔇 Música';
  } catch(e) {
    console.log("El navegador requiere que presiones el botón de música manualmente.");
  }
  setTimeout(startRound,400);
});

restartBtn.addEventListener("click", ()=>{
  endScreen.style.display="none";
  gameContainer.style.display="block";
  currentRound=1;score=0;lives=3;sequenceCount=0;
  updateHUD();
  setTimeout(startRound,400);
});
</script>

</body>
</html>
