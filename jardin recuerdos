<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>El Jardín de los Recuerdos 🌳</title>
<style>
  body {
    font-family: "Comic Sans MS", cursive, sans-serif;
    text-align: center;
    background: url("https://cdn.pixabay.com/photo/2016/11/29/05/08/landscape-1867315_1280.jpg") no-repeat center center fixed;
    background-size: cover;
    color: #fff;
    margin: 0;
  }
  #startScreen, #gameScreen {
    padding: 20px;
  }
  .btn {
    padding: 10px 20px;
    font-size: 18px;
    margin: 10px;
    cursor: pointer;
    border: none;
    border-radius: 10px;
    background: #4CAF50;
    color: white;
  }
  #elements {
    display: grid;
    grid-template-columns: repeat(4, 80px);
    grid-gap: 15px;
    justify-content: center;
    margin-top: 20px;
  }
  .element {
    width: 80px;
    height: 80px;
    font-size: 40px;
    display: flex;
    align-items: center;
    justify-content: center;
    background: rgba(255,255,255,0.8);
    border-radius: 15px;
    cursor: pointer;
    transition: transform 0.2s;
  }
  .element.active {
    transform: scale(1.2);
    background: yellow;
  }
  #gardenerContainer {
    position: fixed;
    bottom: 10px;
    right: 10px;
    display: flex;
    align-items: flex-end;
    z-index: 10;
  }
  #gardener {
    width: 120px;
  }
  #speechBubble {
    max-width: 220px;
    background: rgba(255,255,255,0.9);
    color: #000;
    padding: 10px;
    border-radius: 15px;
    margin-right: 10px;
    font-size: 16px;
    text-align: left;
    display: none;
  }
  #scoreBoard {
    font-size: 20px;
    margin-top: 10px;
  }
</style>
</head>
<body>

<div id="startScreen">
  <h1>🌳 El Jardín de los Recuerdos 🌸</h1>
  <p>Ayuda al jardinero a cuidar su jardín recordando las secuencias.</p>
  <button class="btn" onclick="startGame()">Comenzar Juego</button>
  <button id="musicToggleStart" class="btn">🔊 Música</button>
</div>

<div id="gameScreen" style="display:none;">
  <h2 id="roundTitle">Ronda 1</h2>
  <div id="scoreBoard">Puntos: 0 | Vidas: 🌸🌸🌸</div>
  <div id="elements"></div>
  <button id="musicToggleGame" class="btn">🔊 Música</button>
</div>

<div id="gardenerContainer">
  <div id="speechBubble"></div>
  <img id="gardener" src="https://cdn.pixabay.com/photo/2014/04/03/11/53/gardener-311325_1280.png" alt="Jardinero">
</div>

<audio id="bgMusic" loop src="https://files.freemusicarchive.org/storage-freemusicarchive-org/music/ccCommunity/Jahzzar/Traveller/Jahzzar_-_05_-_Siesta.mp3"></audio>
<audio id="correctSound" src="https://cdn.pixabay.com/download/audio/2022/03/15/audio_9f5c8b5c39.mp3?filename=correct-2-46134.mp3"></audio>
<audio id="wrongSound" src="https://cdn.pixabay.com/download/audio/2021/09/01/audio_99e6892bb2.mp3?filename=error-126627.mp3"></audio>

<script>
const icons = ["🌹","🌻","🌷","🌼","🍎","🍊","🍐","🍓","🥕","🍆","🌽","🍒"];
let sequence = [];
let playerSequence = [];
let round = 1;
let score = 0;
let lives = 3;
let subRound = 1; // controla las 2 secuencias por ronda
const gardenerMessages = [
  "¡Muy bien! El jardín florece gracias a ti 🌸",
  "Atento a la próxima secuencia 👀",
  "¡Excelente memoria! 🌳",
  "No pasa nada, intenta de nuevo 💪",
  "¡El jardín está hermoso gracias a tu ayuda! 🌼"
];

const bgMusic = document.getElementById("bgMusic");
const correctSound = document.getElementById("correctSound");
const wrongSound = document.getElementById("wrongSound");
const speechBubble = document.getElementById("speechBubble");

document.getElementById("musicToggleStart").onclick = toggleMusic;
document.getElementById("musicToggleGame").onclick = toggleMusic;

function toggleMusic(){
  if(bgMusic.paused){
    bgMusic.play();
    document.getElementById("musicToggleStart").textContent="🔇 Música";
    document.getElementById("musicToggleGame").textContent="🔇 Música";
  } else {
    bgMusic.pause();
    document.getElementById("musicToggleStart").textContent="🔊 Música";
    document.getElementById("musicToggleGame").textContent="🔊 Música";
  }
}

function showMessage(text){
  speechBubble.textContent=text;
  speechBubble.style.display="block";
  setTimeout(()=>{ speechBubble.style.display="none"; },5000); // ⏳ mensajes más largos
}

function startGame(){
  document.getElementById("startScreen").style.display="none";
  document.getElementById("gameScreen").style.display="block";
  score=0; lives=3; round=1; subRound=1;
  updateScoreBoard();
  nextRound();
}

function updateScoreBoard(){
  document.getElementById("scoreBoard").textContent=`Puntos: ${score} | Vidas: ${"🌸".repeat(lives)}`;
}

function nextRound(){
  document.getElementById("roundTitle").textContent=`Ronda ${round} - Secuencia ${subRound}/2`;
  sequence=[];
  for(let i=0;i<round+1;i++) sequence.push(Math.floor(Math.random()*icons.length));
  playerSequence=[];
  renderElements();
  showMessage("Observa con atención la secuencia...");
  setTimeout(()=>{ playSequence(sequence); },1000);
}

function renderElements(){
  const container=document.getElementById("elements");
  container.innerHTML="";
  for(let i=0;i<icons.length;i++){
    const div=document.createElement("div");
    div.className="element";
    div.textContent=icons[i];
    div.dataset.index=i;
    // ✅ corrección falsos negativos: tomar siempre dataset
    div.addEventListener("click", () => {
      handleClick(parseInt(div.dataset.index), div);
    });
    container.appendChild(div);
  }
}

function playSequence(seq){
  let i=0;
  const interval=setInterval(()=>{
    if(i<seq.length){
      const cell=document.querySelector(`.element[data-index='${seq[i]}']`);
      cell.classList.add("active");
      setTimeout(()=>cell.classList.remove("active"),500);
      i++;
    } else {
      clearInterval(interval);
    }
  },1000);
}

function handleClick(index, div){
  playerSequence.push(index);
  div.style.transform="scale(0.9)";
  setTimeout(()=>div.style.transform="scale(1)",200);

  const currentStep=playerSequence.length-1;
  if(playerSequence[currentStep]!==sequence[currentStep]){
    wrongSound.play();
    lives--;
    updateScoreBoard();
    showMessage(gardenerMessages[3]);
    playerSequence=[];
    if(lives<=0){
      alert("Juego terminado. Puntuación final: "+score);
      document.location.reload();
    } else {
      setTimeout(()=>nextRound(),1500);
    }
    return;
  }

  if(playerSequence.length===sequence.length){
    correctSound.play();
    score+=round*10;
    updateScoreBoard();
    showMessage(gardenerMessages[Math.floor(Math.random()*gardenerMessages.length)]);

    if(subRound===1){
      subRound=2;
      setTimeout(()=>nextRound(),2000);
    } else {
      subRound=1;
      round++;
      setTimeout(()=>nextRound(),2000);
    }
  }
}
</script>
</body>
</html>
