<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>El Jardín de los Recuerdos</title>
<style>
  body {
    margin: 0;
    font-family: "Comic Sans MS", cursive;
    background: url('https://cdn.pixabay.com/photo/2015/07/09/22/45/field-838684_1280.jpg') no-repeat center center fixed;
    background-size: cover;
    text-align: center;
    color: #2c2c2c;
  }
  h1 { margin-top: 20px; color:#fff; text-shadow:2px 2px 4px #000; }
  #gameArea { margin:20px auto; max-width:800px; }
  #grid {
    display:grid;
    justify-content:center;
    gap:12px;
    margin:20px auto;
  }
  .cell {
    width:80px; height:80px;
    display:flex; align-items:center; justify-content:center;
    font-size:40px; cursor:pointer;
    background:#fff8; border-radius:12px;
    transition:transform 0.2s, background 0.2s;
  }
  .cell.active { background:#ffe066; transform:scale(1.1); }
  #gardener {
    position:relative; display:inline-block; margin-top:10px;
  }
  #gardener img { width:120px; }
  #speech {
    position:absolute; top:-50px; left:130px;
    background:#fff; padding:10px 15px;
    border-radius:12px; border:2px solid #333;
    max-width:250px; text-align:left;
  }
  #lives, #score { font-size:20px; margin:10px; color:#fff; text-shadow:1px 1px 2px #000; }
  #finalScreen {
    position:fixed;top:0;left:0;right:0;bottom:0;
    background:rgba(0,0,0,0.85);color:#fff;display:none;
    flex-direction:column;align-items:center;justify-content:center;
    font-size:22px;padding:20px;
  }
  button {
    font-size:18px;padding:10px 20px;margin-top:20px;
    border:none;border-radius:8px;cursor:pointer;
    background:#4CAF50;color:white;
  }
</style>
</head>
<body>
  <h1>🌳 El Jardín de los Recuerdos 🌳</h1>
  <div id="gameArea">
    <div id="gardener">
      <img src="https://cdn.pixabay.com/photo/2021/02/18/19/35/gardener-6027144_1280.png" alt="Jardinero">
      <div id="speech">¡Bienvenido al jardín!</div>
    </div>
    <div id="lives"></div>
    <div id="score">Puntos: 0</div>
    <div id="grid"></div>
    <button onclick="startGame()">Iniciar Juego</button>
  </div>

  <div id="finalScreen">
    <h2 id="finalTitle"></h2>
    <p id="finalStats"></p>
    <button onclick="startGame()">Jugar de nuevo</button>
  </div>

  <!-- sonidos -->
  <audio id="sndClick" src="https://actions.google.com/sounds/v1/cartoon/wood_plank_flicks.ogg"></audio>
  <audio id="sndErr" src="https://actions.google.com/sounds/v1/cartoon/metal_thud_and_clang.ogg"></audio>
  <audio id="bgMusic" src="https://cdn.pixabay.com/download/audio/2022/03/15/audio_7b9d9ab47a.mp3?filename=happy-day-112193.mp3" loop></audio>

<script>
const grid=document.getElementById('grid');
const speech=document.getElementById('speech');
const livesDiv=document.getElementById('lives');
const scoreDiv=document.getElementById('score');
const finalScreen=document.getElementById('finalScreen');
const finalTitle=document.getElementById('finalTitle');
const finalStats=document.getElementById('finalStats');
const sndClick=document.getElementById('sndClick');
const sndErr=document.getElementById('sndErr');
const bgMusic=document.getElementById('bgMusic');
sndClick.volume=0.6; sndErr.volume=1.0;

let round=1,sequence=[],playerIndex=0,listening=false,lives=3,score=0,errors=0,corrects=0;
const fruits=["🍎","🍐","🍊","🍇","🍓","🍒","🍉","🥕"];
const roundNames=["Sembrar las frutas","Cosechar las frutas","A juntar la cosecha"];
const roundCells=[4,6,8];

function safePlay(snd){
  snd.pause();
  snd.currentTime=0;
  snd.play().catch(()=>{});
}

function showSpeech(msg,time=2500){
  speech.innerText=msg;
  setTimeout(()=>{ speech.innerText=""; }, time);
}

function updateLives(){
  livesDiv.innerHTML="Vidas: "+ "🌸".repeat(lives);
}
function updateScore(){
  scoreDiv.innerText="Puntos: "+score;
}

function startGame(){
  finalScreen.style.display="none";
  round=1;score=0;lives=3;errors=0;corrects=0;
  updateLives();updateScore();
  bgMusic.play().catch(()=>{});
  nextRound();
}

function nextRound(){
  if(round>3){ endGame(true); return; }
  grid.innerHTML="";
  let n=roundCells[round-1];
  grid.style.gridTemplateColumns=`repeat(${Math.ceil(Math.sqrt(n))},80px)`;
  for(let i=0;i<n;i++){
    const cell=document.createElement('div');
    cell.className='cell';
    cell.innerText=fruits[i];
    cell.dataset.index=i;
    cell.onclick=()=>onCellPressed(i);
    grid.appendChild(cell);
  }
  showSpeech("Ronda "+round+": "+roundNames[round-1],4000);
  setTimeout(startSequence,4200);
}

function startSequence(){
  sequence=[];
  let n=roundCells[round-1];
  for(let s=0;s<2;s++){ // 2 secuencias por ronda
    for(let i=0;i<round+1;i++){ sequence.push(Math.floor(Math.random()*n)); }
  }
  playerIndex=0;
  playSequence(0);
}

function playSequence(i){
  if(i>=sequence.length){ listening=true; return; }
  let idx=sequence[i];
  let cell=grid.querySelector(`[data-index='${idx}']`);
  setTimeout(()=>{
    cell.classList.add('active');
    safePlay(sndClick);
    setTimeout(()=>{ cell.classList.remove('active'); playSequence(i+1); },600);
  },600);
}

function onCellPressed(idx){
  if(!listening) return;
  if(idx!==sequence[playerIndex]){
    safePlay(sndErr);
    errors++;
    listening=false;playerIndex=0;
    loseLife();
    return;
  }
  safePlay(sndClick);
  playerIndex++;
  if(playerIndex===sequence.length){
    corrects++;
    score+=round*10;
    updateScore();
    listening=false;round++;
    setTimeout(nextRound,2000);
  }
}

function loseLife(){
  lives--;
  updateLives();
  showSpeech("¡Perdiste una vida! 🌸",2500);
  if(lives<=0){ endGame(false); }
  else { setTimeout(()=>{ playSequence(0); listening=true; },2000); }
}

function endGame(win){
  grid.innerHTML="";
  finalScreen.style.display="flex";
  finalTitle.innerText=win?"¡Felicidades, terminaste el jardín!":"Juego terminado";
  finalStats.innerHTML=`Puntos: ${score}<br>Rondas completadas: ${round-1}<br>Aciertos: ${corrects}<br>Errores: ${errors}`;
  bgMusic.pause();
}
</script>
</body>
</html>
