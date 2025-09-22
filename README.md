<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Juego de Memoria Campestre</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      background: url("https://cdn.pixabay.com/photo/2015/09/02/12/25/field-918531_1280.jpg") no-repeat center center fixed;
      background-size: cover;
      margin: 0;
      padding: 0;
    }

    header.appbar {
      background: rgba(0, 100, 0, 0.8);
      padding: 15px;
    }

    header.appbar h2 {
      margin: 0;
      font-size: 2.4rem;          /* más grande */
      color: #ffffff;             /* blanco puro */
      text-shadow: 2px 2px 6px #000; /* sombra para contraste */
      background: rgba(34,139,34,0.7); /* verde campestre semitransparente */
      padding: 10px 18px;
      border-radius: 10px;
    }

    .hud {
      display: flex;
      justify-content: space-between;
      align-items: center;
      gap: 12px;
      margin-bottom: 10px;
      font-size: 1.4rem;          /* más grande */
      font-weight: bold;          /* más grueso */
      color: #ffffff;             /* blanco */
      text-shadow: 2px 2px 5px #000; /* contraste sobre el fondo */
      padding: 0 20px;
    }

    #gameContainer {
      max-width: 600px;
      margin: auto;
      background: rgba(255, 255, 255, 0.85);
      padding: 20px;
      border-radius: 15px;
      box-shadow: 0 4px 15px rgba(0,0,0,0.3);
    }

    #grid {
      display: grid;
      grid-template-columns: repeat(4, 80px);
      grid-gap: 12px;
      justify-content: center;
      margin-top: 20px;
    }

    .cell {
      width: 80px;
      height: 80px;
      font-size: 2rem;
      display: flex;
      align-items: center;
      justify-content: center;
      background-color: #e0ffe0;
      border: 2px solid #3b7a3b;
      border-radius: 12px;
      cursor: pointer;
      transition: transform 0.2s, background-color 0.3s;
    }

    .cell:active {
      transform: scale(0.9);
      background-color: #c8f7c5;
    }

    #gardenerContainer {
      display: flex;
      align-items: flex-end;
      justify-content: center;
      margin-top: 20px;
      gap: 15px;
    }

    #gardener {
      width: 120px;
      height: auto;
    }

    #speechBubble {
      background: #fff8dc;
      border: 2px solid #000;
      padding: 14px 20px;
      border-radius: 15px;
      max-width: 320px;
      font-size: 1.4rem;     /* más grande */
      font-weight: bold;     /* negrita */
      text-shadow: 1px 1px 3px #00000040; /* sombra suave */
      position: relative;
      transition: background 0.3s;
    }

    #speechBubble.error {
      background: #ff1a1a;   /* rojo fuerte */
      color: white;
      font-weight: bold;
    }

    #speechBubble::after {
      content: "";
      position: absolute;
      bottom: -15px;
      left: 30px;
      border-width: 15px 15px 0;
      border-style: solid;
      border-color: #fff8dc transparent transparent transparent;
    }

    #speechBubble.error::after {
      border-color: #ff1a1a transparent transparent transparent;
    }

    #startButton {
      padding: 12px 24px;
      font-size: 1.2rem;
      background: #3b7a3b;
      color: white;
      border: none;
      border-radius: 10px;
      cursor: pointer;
      transition: background 0.3s, transform 0.2s;
      margin-top: 15px;
    }

    #startButton:hover {
      background: #2d5a2d;
      transform: scale(1.05);
    }
  </style>
</head>
<body>
  <header class="appbar">
    <h2>🌱 Juego de Memoria Campestre 🌱</h2>
  </header>

  <div id="gameContainer">
    <div class="hud">
      <div id="lives">❤️❤️❤️</div>
      <div id="score">Puntos: 0</div>
    </div>

    <button id="startButton">Iniciar Juego</button>
    <div id="grid"></div>

    <div id="gardenerContainer">
      <img id="gardener" src="https://cdn.pixabay.com/photo/2014/04/03/10/01/farmer-309726_1280.png" alt="jardinero">
      <div id="speechBubble">¡Hola! Presiona "Iniciar Juego".</div>
    </div>
  </div>

  <script>
    const startButton = document.getElementById("startButton");
    const grid = document.getElementById("grid");
    const speechBubble = document.getElementById("speechBubble");
    const livesDisplay = document.getElementById("lives");
    const scoreDisplay = document.getElementById("score");

    const fruits = ["🍎","🍐","🍊","🍌","🍇","🍉","🍓","🍒"];

    const successSound = new Audio("https://actions.google.com/sounds/v1/cartoon/wood_plank_flicks.ogg");
    const errorSound = new Audio("https://actions.google.com/sounds/v1/cartoon/clang_and_wobble.ogg");

    let sequence = [];
    let playerSequence = [];
    let round = 0;
    let lives = 3;
    let score = 0;
    let acceptingInput = false;

    function updateLives() {
      livesDisplay.textContent = "❤️".repeat(lives);
    }

    function updateScore() {
      scoreDisplay.textContent = "Puntos: " + score;
    }

    function showMessage(message, isError = false) {
      speechBubble.textContent = message;
      if (isError) {
        speechBubble.classList.add("error");
        setTimeout(() => {
          speechBubble.classList.remove("error");
          speechBubble.textContent = "Intenta de nuevo la secuencia.";
        }, 3000);
      } else {
        setTimeout(() => {
          speechBubble.textContent = "";
        }, 3000);
      }
    }

    function getRoundMessage(round) {
      if (round === 1) return "🌱 Ronda 1: ¡A sembrar las frutas!";
      if (round === 2) return "🍎 Ronda 2: ¡A cosechar las frutas!";
      if (round === 3) return "🧺 Ronda 3: ¡A juntar la cosecha!";
      return `Ronda ${round}`;
    }

    function startGame() {
      sequence = [];
      playerSequence = [];
      round = 0;
      lives = 3;
      score = 0;
      updateLives();
      updateScore();
      showMessage("¡Comencemos el juego!");
      nextRound();
    }

    function nextRound() {
      round++;
      playerSequence = [];

      let cellsCount = 4;
      if (round === 2) cellsCount = 6;
      if (round >= 3) cellsCount = 8;

      grid.innerHTML = "";
      grid.style.gridTemplateColumns = `repeat(${cellsCount}, 80px)`;
      for (let i = 0; i < cellsCount; i++) {
        const cell = document.createElement("div");
        cell.classList.add("cell");
        cell.textContent = fruits[i % fruits.length];
        cell.addEventListener("click", () => handlePlayerInput(i));
        grid.appendChild(cell);
      }

      sequence.push(Math.floor(Math.random() * cellsCount));
      showMessage(getRoundMessage(round));
      playSequence();
    }

    function playSequence() {
      acceptingInput = false;
      let i = 0;
      const interval = setInterval(() => {
        const index = sequence[i];
        const cell = grid.children[index];
        cell.style.backgroundColor = "#ffff99";
        successSound.play();
        setTimeout(() => {
          cell.style.backgroundColor = "#e0ffe0";
        }, 500);
        i++;
        if (i >= sequence.length) {
          clearInterval(interval);
          acceptingInput = true;
        }
      }, 1000);
    }

    function handlePlayerInput(index) {
      if (!acceptingInput) return;
      const cell = grid.children[index];
      playerSequence.push(index);

      cell.style.backgroundColor = "#add8e6";
      successSound.play();
      setTimeout(() => {
        cell.style.backgroundColor = "#e0ffe0";
      }, 300);

      if (playerSequence[playerSequence.length - 1] !== sequence[playerSequence.length - 1]) {
        lives--;
        updateLives();
        errorSound.play();
        if (lives <= 0) {
          showMessage("😢 ¡Te has quedado sin vidas! Fin del juego.", true);
          acceptingInput = false;
          return;
        }
        showMessage("❌ Te equivocaste. Intenta de nuevo.", true);
        playerSequence = [];
        setTimeout(playSequence, 1500);
        return;
      }

      if (playerSequence.length === sequence.length) {
        score += 10;
        updateScore();
        setTimeout(nextRound, 1500);
      }
    }

    startButton.addEventListener("click", startGame);
  </script>
</body>
</html>
