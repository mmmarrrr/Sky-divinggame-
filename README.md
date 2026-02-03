<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>SkyDive 4000 - Real Adventure</title>
    <style>
        body { margin: 0; overflow: hidden; background: #87CEEB; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; touch-action: none; }
        #ui { position: absolute; top: 15px; left: 15px; color: white; text-shadow: 2px 2px 4px #000; pointer-events: none; z-index: 10; }
        
        /* Message Screens */
        .overlay { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); 
                   background: rgba(255,255,255,0.98); padding: 30px; border-radius: 20px; text-align: center; 
                   box-shadow: 0 10px 30px rgba(0,0,0,0.5); width: 85%; max-width: 450px; z-index: 20; max-height: 90vh; overflow-y: auto; }
        
        h1 { color: #2980b9; margin-top: 0; }
        .marketing-text { font-style: italic; color: #444; line-height: 1.5; margin: 15px 0; font-size: 15px; }
        .cta-text { font-weight: bold; color: #2c3e50; border: 2px solid #2ecc71; padding: 10px; border-radius: 10px; margin-top: 15px; }
        
        input { padding: 12px; margin: 10px 0; border: 2px solid #3498db; border-radius: 8px; width: 90%; font-size: 16px; }
        button { padding: 15px 30px; background: #2ecc71; color: white; border: none; border-radius: 8px; 
                 cursor: pointer; font-weight: bold; font-size: 18px; width: 100%; transition: transform 0.1s; }
        button:active { transform: scale(0.95); }
        
        .hidden { display: none; }
        canvas { display: block; background: linear-gradient(#1e90ff, #87CEEB); }
        
        /* Leaderboard */
        #leaderboard { margin: 15px 0; text-align: left; background: #f9f9f9; padding: 10px; border-radius: 8px; }
        ul { list-style: none; padding: 0; margin: 0; }
        li { padding: 5px 0; border-bottom: 1px solid #ddd; display: flex; justify-content: space-between; font-size: 14px; }
    </style>
</head>
<body>

    <div id="ui">
        <div style="font-size: 20px;">Altitude: <span id="alt">4000</span>m</div>
        <div style="font-size: 16px;">Survival Time: <span id="timer">0.0</span>s</div>
    </div>

    <div id="menu" class="overlay">
        <h1>🪂 SkyDive 4000</h1>
        <p class="marketing-text">"Get ready to fly! You’re about to experience the ultimate virtual drop. Feel the rush, master the winds, and find your courage—because this is just the warm-up for the real thing. Ready to jump?"</p>
        <input type="text" id="playerName" placeholder="Enter Your Name">
        <button onclick="startGame(true)">START JUMP</button>
    </div>

    <div id="end-screen" class="overlay hidden">
        <h1 id="status-title">Nice Try!</h1>
        <p style="font-size: 1.2em;">Survival Time: <span id="finalTime" style="font-weight:bold; color:#e67e22;"></span>s</p>
        
        <p class="cta-text">"Nailed the landing? Now do it for real! Don't just play the adventure—live it at our company. Our world-class professionals and top-tier safety gear make the transition from screen to sky seamless and exhilarating. Stop gaming and start flying; book your real-life jump with our experts today!"</p>
        
        <div id="leaderboard">
            <strong style="display:block; margin-bottom:5px;">🏆 Survival Rankings:</strong>
            <div id="scores-list"></div>
        </div>
        
        <button onclick="startGame(false)">FLY AGAIN</button>
    </div>

    <canvas id="gameCanvas"></canvas>

<script>
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
let gameActive = false;
let altitude = 4000;
let startTime, playerName = "";
let player = { x: 0, y: 120, w: 50, h: 50 };
let obstacles = [];
let frameCount = 0;

function resize() {
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
    player.x = canvas.width / 2 - 25;
}
window.addEventListener('resize', resize);
resize();

// Touch & Mouse Support
const handleMove = (e) => {
    if (!gameActive) return;
    const clientX = (e.touches ? e.touches[0].clientX : e.clientX);
    player.x = clientX - player.w/2;
    if (player.x < 0) player.x = 0;
    if (player.x > canvas.width - player.w) player.x = canvas.width - player.w;
};
canvas.addEventListener('mousemove', handleMove);
canvas.addEventListener('touchmove', (e) => { e.preventDefault(); handleMove(e); }, {passive: false});

function startGame(isFirstTime) {
    if (isFirstTime) {
        const nameInput = document.getElementById('playerName').value;
        if (!nameInput) return alert("Please enter your name to log your score!");
        playerName = nameInput;
    }
    
    document.getElementById('menu').classList.add('hidden');
    document.getElementById('end-screen').classList.add('hidden');
    
    altitude = 4000;
    obstacles = [];
    frameCount = 0;
    startTime = Date.now();
    gameActive = true;
    requestAnimationFrame(animate);
}

function update() {
    altitude -= 1.5;
    const currentSurvival = ((Date.now() - startTime) / 1000).toFixed(1);
    document.getElementById('timer').innerText = currentSurvival;
    document.getElementById('alt').innerText = Math.max(0, Math.floor(altitude));

    if (altitude <= 0) { endGame("SAFE LANDING!"); return; }

    // Increase difficulty over time
    let spawnRate = Math.max(7, 45 - Math.floor((4000 - altitude) / 90)); 
    if (frameCount % spawnRate === 0) {
        obstacles.push({
            x: Math.random() * (canvas.width - 50),
            y: canvas.height,
            symbol: Math.random() > 0.4 ? '🦅' : '🚁',
            speed: 5 + Math.random() * 7
        });
    }

    for (let i = obstacles.length - 1; i >= 0; i--) {
        let o = obstacles[i];
        o.y -= o.speed;
        
        if (player.x < o.x + 35 && player.x + player.w > o.x &&
            player.y < o.y + 35 && player.y + player.h > o.y) {
            endGame("CRASHED!");
            return;
        }
        if (o.y < -50) obstacles.splice(i, 1);
    }
}

function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    ctx.font = "50px serif";
    ctx.fillText("🪂", player.x, player.y + 40);
    ctx.font = "40px serif";
    obstacles.forEach(o => ctx.fillText(o.symbol, o.x, o.y));
}

function animate() {
    if (!gameActive) return;
    frameCount++;
    update();
    draw();
    if (gameActive) requestAnimationFrame(animate);
}

function endGame(status) {
    gameActive = false;
    const finalSurvival = ((Date.now() - startTime) / 1000).toFixed(2);
    document.getElementById('status-title').innerText = status;
    document.getElementById('finalTime').innerText = finalSurvival;
    document.getElementById('end-screen').classList.remove('hidden');
    saveScore(playerName, finalSurvival);
    displayLeaderboard();
}

function saveScore(name, time) {
    let scores = JSON.parse(localStorage.getItem('skyDiveSurvival') || "[]");
    scores.push({ name, time: parseFloat(time) });
    scores.sort((a, b) => b.time - a.time);
    localStorage.setItem('skyDiveSurvival', JSON.stringify(scores.slice(0, 5)));
}

function displayLeaderboard() {
    let scores = JSON.parse(localStorage.getItem('skyDiveSurvival') || "[]");
    let html = "<ul>" + scores.map((s, i) => 
        `<li><span>${i+1}. ${s.name}</span> <b>${s.time}s</b></li>`
    ).join('') + "</ul>";
    document.getElementById('scores-list').innerHTML = html;
}
</script>
</body>
</html>
