<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>SkyDive 4000m</title>
    <style>
        body { margin: 0; overflow: hidden; background: #87CEEB; font-family: sans-serif; touch-action: none; }
        #ui { position: absolute; top: 10px; left: 10px; color: white; text-shadow: 2px 2px 4px #000; pointer-events: none; }
        #menu, #win-screen { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); 
                   background: rgba(255,255,255,0.9); padding: 20px; border-radius: 10px; text-align: center; }
        input { padding: 10px; margin: 10px; border: 1px solid #ccc; border-radius: 5px; }
        button { padding: 10px 20px; background: #2ecc71; color: white; border: none; border-radius: 5px; cursor: pointer; }
        .hidden { display: none; }
        canvas { display: block; background: linear-gradient(#1e90ff, #87CEEB); }
    </style>
</head>
<body>

    <div id="ui">
        <div id="stats">Altitude: <span id="alt">4000</span>m | Speed: <span id="speed">0</span>km/h</div>
    </div>

    <div id="menu">
        <h1>SkyDive 4000</h1>
        <p>Avoid Eagles and Helicopters!</p>
        <input type="text" id="playerName" placeholder="Enter your name">
        <br>
        <button onclick="startGame()">Start Jump</button>
    </div>

    <div id="win-screen" class="hidden">
        <h1 id="win-msg">Safe Landing!</h1>
        <p>Final Time: <span id="finalTime"></span>s</p>
        <h3>Rankings:</h3>
        <div id="leaderboard"></div>
        <button onclick="resetGame()">Dive Again</button>
    </div>

    <canvas id="gameCanvas"></canvas>

<script>
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
let gameActive = false;
let altitude = 4000;
let startTime, playerName = "";
let player = { x: 0, y: 150, w: 40, h: 60 };
let obstacles = [];
let frameCount = 0;

function resize() {
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
    player.x = canvas.width / 2 - 20;
}
window.addEventListener('resize', resize);
resize();

// Touch/Mouse Controls
window.addEventListener('pointermove', (e) => {
    if (gameActive) player.x = e.clientX - player.w/2;
});

function startGame() {
    const nameInput = document.getElementById('playerName').value;
    if (!nameInput) return alert("Please enter a name");
    playerName = nameInput;
    document.getElementById('menu').classList.add('hidden');
    document.getElementById('win-screen').classList.add('hidden');
    resetStats();
    gameActive = true;
    startTime = Date.now();
    animate();
}

function resetStats() {
    altitude = 4000;
    obstacles = [];
    frameCount = 0;
}

function spawnObstacle() {
    // Difficulty increases as altitude drops
    let spawnRate = Math.max(10, 60 - (4000 - altitude) / 100); 
    if (frameCount % Math.floor(spawnRate) === 0) {
        let type = Math.random() > 0.5 ? '🦅' : '🚁';
        obstacles.push({
            x: Math.random() * (canvas.width - 40),
            y: canvas.height,
            symbol: type,
            speed: 5 + (Math.random() * 5)
        });
    }
}

function update() {
    altitude -= 1.5; // Constant descent
    if (altitude <= 0) endGame(true);

    spawnObstacle();

    obstacles.forEach((obs, index) => {
        obs.y -= obs.speed; // Obstacles "rise" relative to falling player
        if (obs.y < -50) obstacles.splice(index, 1);

        // Simple Collision
        if (player.x < obs.x + 30 && player.x + player.w > obs.x &&
            player.y < obs.y + 30 && player.y + player.h > obs.y) {
            endGame(false);
        }
    });

    document.getElementById('alt').innerText = Math.max(0, Math.floor(altitude));
    document.getElementById('speed').innerText = Math.floor(180 + Math.random() * 10);
}

function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    
    // Draw Player (Diver)
    ctx.font = "50px serif";
    ctx.fillText("🧍", player.x, player.y + 40);

    // Draw Obstacles
    ctx.font = "40px serif";
    obstacles.forEach(obs => {
        ctx.fillText(obs.symbol, obs.x, obs.y);
    });
}

function animate() {
    if (!gameActive) return;
    frameCount++;
    update();
    draw();
    requestAnimationFrame(animate);
}

function endGame(win) {
    gameActive = false;
    const timeTaken = ((Date.now() - startTime) / 1000).toFixed(2);
    document.getElementById('win-screen').classList.remove('hidden');
    
    if (win) {
        document.getElementById('win-msg').innerText = "Safe Landing!";
        saveScore(playerName, timeTaken);
    } else {
        document.getElementById('win-msg').innerText = "Collision! Try again.";
    }
    
    document.getElementById('finalTime').innerText = win ? timeTaken : "N/A";
    displayLeaderboard();
}

function saveScore(name, time) {
    let scores = JSON.parse(localStorage.getItem('skyDiveScores') || "[]");
    scores.push({ name, time: parseFloat(time) });
    scores.sort((a, b) => a.time - b.time); // Faster is better
    localStorage.setItem('skyDiveScores', JSON.stringify(scores.slice(0, 5)));
}

function displayLeaderboard() {
    let scores = JSON.parse(localStorage.getItem('skyDiveScores') || "[]");
    let html = "<ul>" + scores.map(s => `<li>${s.name}: ${s.time}s</li>`).join('') + "</ul>";
    document.getElementById('leaderboard').innerHTML = html;
}

function resetGame() {
    document.getElementById('win-screen').classList.add('hidden');
    resetStats();
    gameActive = true;
    startTime = Date.now();
    animate();
}
</script>
</body>
</html>
