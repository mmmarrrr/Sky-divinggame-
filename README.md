<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>SkyDive 4000 - Global Leader</title>
    <script src="https://www.gstatic.com/firebasejs/9.22.1/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.22.1/firebase-database-compat.js"></script>
    <style>
        body { margin: 0; overflow: hidden; background: #87CEEB; font-family: 'Segoe UI', sans-serif; touch-action: none; }
        #ui { position: absolute; top: 15px; left: 15px; color: white; text-shadow: 2px 2px 4px #000; pointer-events: none; z-index: 10; }
        #global-record { position: absolute; top: 15px; right: 15px; background: rgba(0,0,0,0.6); color: #f1c40f; padding: 10px; border-radius: 8px; font-weight: bold; }
        .overlay { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); 
                   background: rgba(255,255,255,0.98); padding: 30px; border-radius: 20px; text-align: center; 
                   box-shadow: 0 10px 30px rgba(0,0,0,0.5); width: 85%; max-width: 450px; z-index: 20; }
        .marketing-text { font-style: italic; color: #444; font-size: 14px; margin: 15px 0; line-height: 1.4; }
        input { padding: 12px; margin: 10px 0; border: 2px solid #3498db; border-radius: 8px; width: 90%; }
        button { padding: 15px; background: #2ecc71; color: white; border: none; border-radius: 8px; cursor: pointer; font-weight: bold; width: 100%; margin-top: 10px; }
        .hidden { display: none; }
        canvas { display: block; }
    </style>
</head>
<body>

    <div id="ui">
        <div>Altitude: <span id="alt">4000</span>m</div>
        <div>Time: <span id="timer">0.0</span>s</div>
    </div>

    <div id="global-record">🏆 World Record: <span id="top-score">0.00</span>s <br>by <span id="top-name">None</span></div>

    <div id="menu" class="overlay">
        <h1>🪂 SkyDive 4000</h1>
        <p class="marketing-text">"Get ready to fly! ... Ready to jump?"</p>
        <input type="text" id="playerName" placeholder="Enter Your Name">
        <button onclick="startGame(true)">START JUMP</button>
    </div>

    <div id="end-screen" class="overlay hidden">
        <h1 id="status-title">CRASHED!</h1>
        <p>Your Survival Time: <span id="finalTime" style="font-weight:bold; color:#e67e22;"></span>s</p>
        <p class="marketing-text">"Nailed the landing? Now do it for real! ... Stop gaming and start flying!"</p>
        <button onclick="startGame(false)">FLY AGAIN</button>
    </div>

    <canvas id="gameCanvas"></canvas>

<script>
// --- FIREBASE CONFIGURATION ---
// REPLACE THIS WITH YOUR CODE FROM FIREBASE CONSOLE
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  databaseURL: "https://YOUR_PROJECT.firebaseio.com",
  projectId: "YOUR_PROJECT",
  storageBucket: "YOUR_PROJECT.appspot.com",
  messagingSenderId: "YOUR_ID",
  appId: "YOUR_APP_ID"
};

firebase.initializeApp(firebaseConfig);
const database = firebase.database();

// --- GAME LOGIC ---
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
let gameActive = false, altitude = 4000, startTime, playerName = "", obstacles = [], frameCount = 0;
let player = { x: 0, y: 120, w: 50, h: 50 };

function resize() {
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
    player.x = canvas.width / 2 - 25;
}
window.addEventListener('resize', resize);
resize();

// Global Score Sync
database.ref('highScore').on('value', (snapshot) => {
    const data = snapshot.val();
    if (data) {
        document.getElementById('top-score').innerText = data.time;
        document.getElementById('top-name').innerText = data.name;
    }
});

function startGame(isFirstTime) {
    if (isFirstTime) {
        playerName = document.getElementById('playerName').value || "Anonymous";
    }
    document.getElementById('menu').classList.add('hidden');
    document.getElementById('end-screen').classList.add('hidden');
    altitude = 4000; obstacles = []; frameCount = 0; startTime = Date.now();
    gameActive = true;
    requestAnimationFrame(animate);
}

function endGame(status) {
    gameActive = false;
    const finalTime = parseFloat(((Date.now() - startTime) / 1000).toFixed(2));
    document.getElementById('status-title').innerText = status;
    document.getElementById('finalTime').innerText = finalTime;
    document.getElementById('end-screen').classList.remove('hidden');

    // Check if it's the new world record
    database.ref('highScore').once('value').then((snapshot) => {
        const currentTop = snapshot.val() ? snapshot.val().time : 0;
        if (finalTime > currentTop) {
            database.ref('highScore').set({ name: playerName, time: finalTime });
        }
    });
}

// (Input and Update logic remains the same as previous versions...)
const handleMove = (e) => {
    if (!gameActive) return;
    const clientX = (e.touches ? e.touches[0].clientX : e.clientX);
    player.x = clientX - player.w/2;
};
canvas.addEventListener('mousemove', handleMove);
canvas.addEventListener('touchmove', (e) => { e.preventDefault(); handleMove(e); }, {passive: false});

function update() {
    altitude -= 1.5;
    document.getElementById('timer').innerText = ((Date.now() - startTime) / 1000).toFixed(1);
    document.getElementById('alt').innerText = Math.max(0, Math.floor(altitude));
    if (altitude <= 0) { endGame("SAFE LANDING!"); return; }
    let spawnRate = Math.max(7, 45 - Math.floor((4000-altitude)/90));
    if (frameCount++ % spawnRate === 0) {
        obstacles.push({ x: Math.random()*(canvas.width-50), y: canvas.height, symbol: Math.random()>0.4?'🦅':'🚁', speed: 5+Math.random()*7 });
    }
    for (let i = obstacles.length - 1; i >= 0; i--) {
        obstacles[i].y -= obstacles[i].speed;
        if (player.x < obstacles[i].x + 35 && player.x + player.w > obstacles[i].x && player.y < obstacles[i].y + 35 && player.y + player.h > obstacles[i].y) {
            endGame("CRASHED!"); return;
        }
        if (obstacles[i].y < -50) obstacles.splice(i, 1);
    }
}

function animate() {
    if (!gameActive) return;
    update();
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    ctx.font = "50px serif"; ctx.fillText("🪂", player.x, player.y + 40);
    ctx.font = "40px serif"; obstacles.forEach(o => ctx.fillText(o.symbol, o.x, o.y));
    requestAnimationFrame(animate);
}
</script>
</body>
</html>
