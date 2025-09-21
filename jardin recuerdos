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
    background: url("https://upload.wikimedia.org/wikipedia/commons/6/6e/Field_in_summer.JPG") no-repeat center center fixed;
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
    grid-template-columns: repeat(4,90px);
    grid-gap:15px;
    margin-top:100px;
  }
  .cell {
    width:90px; height:90px;
    display:flex; align-items:center; justify-content:center;
    font-size:2.5em;
    background:white;
    border-radius:15px;
    box-shadow:2px 2px 5px rgba(0,0,0,0.3);
    cursor:pointer;
    transition:transform 0.2s, background 0.2s;
  }
  .highlight {
    background:yellow;
    transform:scale(1.15);
  }
  #gardener {
    position:absolute; bottom:20px; left:20px;
    width:200px;
    text-align:center;
  }
  #gardener span {
    font-size:3em;
    display:block;
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
    <div id="lives">🌸🌸🌸</div>
    <button id="musicBtnGame">🔊 Música</button>
  </div>
  <div id="grid"></div>
  <div id="gardener">
    <span>👨‍🌾</span>
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
const items = ["🍎","🍌","🍇","🥕","🍓","🍊","🥦"];
let sequence = [];
let playerSequence = [];
let round = 1;
let score = 0;
let lives = 3;
let playingSequence = false;
let sequencePerRound = 0;

const grid = document.getElementById("grid");
const speech = document.getElementById("speech");
const roundDisplay = document.getElementById("round");
const scoreDisplay = document.getElementById("score");
const livesDisplay = document.getElementById("lives");

const bgMusic = document.getElementById("bgMusic");
const sounds = [document.getElementById("sound1"), document.getElementById("sound2"), document.getElementById("sound3")];

document.getElementById("startBtn").addEventListener("click", startGame);
document.getElementById("restartBtn").addEventListener("click", () => location.reload());
document.getElementById("musicBtnStart").addEventListener("click", () => toggleMusic());
document.getElementById("musicBtnGame").addEventListener("click", () => toggleMusic());

function toggleMusic() {
  if (bgMusic.paused) bgMusic.play();
  else bgMusic.pause();
}

function startGame() {
  document.getElementById("startScreen").style.display = "none";
  document.getElementById("gameContainer").style.display = "flex";
  round = 1;
  score = 0;
  lives = 3;
  sequencePerRound = 0;
  updateHUD();
  nextSequence();
}

function updateHUD() {
  roundDisplay.textContent = "Ronda: " + round;
  scoreDisplay.textContent = "Puntos: " + score;
  livesDisplay.textContent = "🌸".repeat(lives);
}

function nextSequence() {
  playingSequence = true;
  playerSequence = [];
  const nextItem = items[Math.floor(Math.random()*items.length)];
  sequence.push(nextItem);
  sequencePerRound++;

  showGardenerMessage();

  let i = 0;
  const interval = setInterval(()=>{
    if (i < sequence.length) {
      const cell = Array.from(grid.children).find(c => c.textContent === sequence[i]);
      if (cell) {
        cell.classList.add("highlight");
        sounds[i % sounds.length].play();
        setTimeout(()=>cell.classList.remove("highlight"), 600);
      }
      i++;
    } else {
      clearInterval(interval);
      playingSequence = false;
      speech.textContent = "¡Es tu turno!";
    }
  }, 1000);
}

function checkChoice(item) {
  if (playingSequence) return;
  playerSequence.push(item);

  const index = playerSequence.length-1;
  if (playerSequence[index] !== sequence[index]) {
    lives--;
    updateHUD();
    if (lives <= 0) return endGame();
    speech.textContent = "¡Ups! Pierdes una flor 🌸";
    playerSequence = [];
    return;
  }

  sounds[index % sounds.length].play();

  if (playerSequence.length === sequence.length) {
    score += 10;
    updateHUD();
    if (sequencePerRound < 2) {
      speech.textContent = "¡Muy bien! Ahora otra secuencia 🌱";
      setTimeout(()=> nextSequence(), 2000);
    } else {
      round++;
      sequence = [];
      sequencePerRound = 0;
      if (round > 3) return endGame();
      setTimeout(()=> nextSequence(), 3000);
    }
  }
}

function endGame() {
  document.getElementById("gameContainer").style.display = "none";
  document.getElementById("endScreen").style.display = "flex";
  document.getElementById("finalScore").textContent = "Tu puntuación: " + score;
}

function showGardenerMessage() {
  if (round === 1) speech.textContent = "🌱 Ronda 1: ¡A sembrar las frutas!";
  else if (round === 2) speech.textContent = "🍎 Ronda 2: ¡Hora de cosechar!";
  else if (round === 3) speech.textContent = "🥕 Ronda 3: ¡A juntar la cosecha!";
}

function createGrid() {
  grid.innerHTML = "";
  items.forEach(item=>{
    const cell = document.createElement("div");
    cell.classList.add("cell");
    cell.textContent = item;
    cell.addEventListener("click", ()=> checkChoice(item));
    grid.appendChild(cell);
  });
}
createGrid();
</script>
</body>
</html>
