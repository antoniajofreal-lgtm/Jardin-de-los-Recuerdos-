<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>El Jardín de los Recuerdos 🌳</title>
  <style>
    body {
      margin: 0;
      font-family: 'Comic Sans MS', cursive, sans-serif;
      background: url("https://upload.wikimedia.org/wikipedia/commons/6/6e/Field_in_summer.JPG") no-repeat center center fixed;
      background-size: cover;
      overflow: hidden;
      text-align: center;
      color: #2e2a2a;
    }

    h1 {
      font-size: 2.5em;
      margin-top: 10px;
      text-shadow: 2px 2px 4px #fff;
    }

    #game {
      margin-top: 20px;
    }

    .grid {
      display: grid;
      grid-template-columns: repeat(4, 100px);
      grid-gap: 15px;
      justify-content: center;
      margin-top: 20px;
    }

    .cell {
      width: 100px;
      height: 100px;
      font-size: 2.5em;
      display: flex;
      align-items: center;
      justify-content: center;
      border-radius: 15px;
      background: rgba(255,255,255,0.7);
      cursor: pointer;
      transition: transform 0.2s, background 0.2s;
    }

    .cell.active {
      background: yellow;
      transform: scale(1.1);
    }

    #gardener {
      margin-top: 20px;
      font-size: 2em;
    }

    #dialogue {
      margin-top: 10px;
      background: rgba(255, 255, 255, 0.8);
      display: inline-block;
      padding: 10px 20px;
      border-radius: 15px;
      font-size: 1.2em;
      max-width: 80%;
    }

    #scoreboard {
      margin-top: 20px;
      font-size: 1.2em;
    }

    #lives {
      margin-top: 10px;
      font-size: 1.5em;
    }

    #startBtn {
      margin-top: 20px;
      padding: 10px 20px;
      font-size: 1.2em;
      border: none;
      border-radius: 10px;
      background: #6ab04c;
      color: white;
      cursor: pointer;
    }

    #startBtn:hover {
      background: #218c74;
    }
  </style>
</head>
<body>
  <h1>🌳 El Jardín de los Recuerdos 🌸</h1>

  <div id="game">
    <button id="startBtn">Iniciar Juego</button>
    <div id="gardener">👨‍🌾</div>
    <div id="dialogue">¡Bienvenido al jardín! 🌼</div>

    <div id="scoreboard">
      Puntuación: <span id="score">0</span>
    </div>
    <div id="lives">🌸🌸🌸</div>

    <div class="grid" id="grid"></div>
  </div>

  <audio id="sound1" src="https://actions.google.com/sounds/v1/cartoon/pop.ogg"></audio>
  <audio id="sound2" src="https://actions.google.com/sounds/v1/cartoon/wood_plank_flicks.ogg"></audio>
  <audio id="sound3" src="https://actions.google.com/sounds/v1/cartoon/clang_and_wobble.ogg"></audio>
  <audio id="sound4" src="https://actions.google.com/sounds/v1/cartoon/slide_whistle_to_drum.ogg"></audio>

  <script>
    const icons = ["🍎", "🍌", "🥕", "🍓", "🍊", "🍇", "🥦", "🍉"];
    const sounds = [
      document.getElementById("sound1"),
      document.getElementById("sound2"),
      document.getElementById("sound3"),
      document.getElementById("sound4")
    ];

    const gardener = document.getElementById("gardener");
    const dialogue = document.getElementById("dialogue");
    const grid = document.getElementById("grid");
    const scoreSpan = document.getElementById("score");
    const livesDiv = document.getElementById("lives");
    const startBtn = document.getElementById("startBtn");

    let sequence = [];
    let playerSequence = [];
    let score = 0;
    let lives = 3;
    let round = 1;
    let showing = false;
    let seqPerRound = 2;
    let currentSeqCount = 0;

    // Crear casillas
    icons.forEach((icon, index) => {
      const cell = document.createElement("div");
      cell.classList.add("cell");
      cell.textContent = icon;
      cell.dataset.index = index;
      cell.addEventListener("click", () => handlePlayerClick(index));
      grid.appendChild(cell);
    });

    function playSound(index) {
      const s = sounds[index % sounds.length];
      s.currentTime = 0;
      s.play();
    }

    function highlightCell(index) {
      const cell = grid.querySelector(`.cell[data-index='${index}']`);
      cell.classList.add("active");
      playSound(index);
      setTimeout(() => {
        cell.classList.remove("active");
      }, 600);
    }

    function setDialogue(text, duration = 2000) {
      dialogue.textContent = text;
      if (duration > 0) {
        setTimeout(() => {
          dialogue.textContent = "";
        }, duration);
      }
    }

    function updateLives() {
      livesDiv.textContent = "🌸".repeat(lives);
    }

    function startGame() {
      score = 0;
      lives = 3;
      round = 1;
      currentSeqCount = 0;
      updateLives();
      scoreSpan.textContent = score;
      setDialogue("¡Vamos a cuidar el jardín! 🌼", 3000);
      nextRound();
    }

    function nextRound() {
      if (lives <= 0) {
        setDialogue("Juego terminado 💐", 3000);
        return;
      }

      if (round === 1) setDialogue("Ronda 1: 🌱 ¡A sembrar las frutas!", 4000);
      if (round === 2) setDialogue("Ronda 2: 🍎 ¡Hora de cosechar!", 4000);
      if (round === 3) setDialogue("Ronda 3: 🥕 ¡A juntar la cosecha!", 4000);

      currentSeqCount = 0;
      newSequence();
    }

    function newSequence() {
      playerSequence = [];
      sequence = [];
      for (let i = 0; i < round + 2; i++) {
        sequence.push(Math.floor(Math.random() * icons.length));
      }
      showSequence();
    }

    function showSequence() {
      showing = true;
      let i = 0;
      const interval = setInterval(() => {
        highlightCell(sequence[i]);
        i++;
        if (i >= sequence.length) {
          clearInterval(interval);
          showing = false;
          setDialogue("¡Tu turno! 🌸", 2000);
        }
      }, 1000);
    }

    function handlePlayerClick(index) {
      if (showing) return;

      playSound(index);
      playerSequence.push(index);

      const currentStep = playerSequence.length - 1;
      if (playerSequence[currentStep] !== sequence[currentStep]) {
        lives--;
        updateLives();
        setDialogue("Ups, intenta de nuevo 🌼", 3000);
        if (lives > 0) {
          playerSequence = [];
          setTimeout(showSequence, 2000);
        } else {
          setDialogue("Juego terminado 💐", 3000);
        }
        return;
      }

      if (playerSequence.length === sequence.length) {
        score += sequence.length * 10;
        scoreSpan.textContent = score;
        currentSeqCount++;

        if (currentSeqCount < seqPerRound) {
          setDialogue("¡Muy bien! 🌻 Vamos con otra secuencia", 3000);
          setTimeout(newSequence, 3000);
        } else {
          round++;
          if (round > 3) {
            setDialogue("¡Felicidades, completaste el jardín! 🌺", 4000);
          } else {
            setTimeout(nextRound, 4000);
          }
        }
      }
    }

    startBtn.addEventListener("click", startGame);
  </script>
</body>
</html>
