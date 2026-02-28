<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Traffic Rule Racer</title>
    <style>
        :root {
            --bg-color: #f0f2f5;
            --road-color: #333;
            --ui-panel: rgba(255, 255, 255, 0.9);
            --accent: #3498db;
        }

        body {
            margin: 0;
            padding: 0;
            overflow: hidden;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: var(--bg-color);
            touch-action: none;
        }

        #game-container {
            position: relative;
            width: 100vw;
            height: 100vh;
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        /* Top UI Bar */
        #top-ui {
            position: absolute;
            top: 10px;
            width: 90%;
            max-width: 500px;
            height: 50px;
            background: var(--ui-panel);
            border-radius: 12px;
            display: flex;
            justify-content: space-around;
            align-items: center;
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
            z-index: 10;
        }

        .ui-item { text-align: center; font-weight: bold; font-size: 14px; }
        .ui-label { font-size: 10px; color: #666; display: block; }

        /* Game Canvas */
        canvas {
            display: block;
            background: #4caf50; /* Grass */
        }

        /* Controls */
        #controls {
            position: absolute;
            bottom: 30px;
            width: 100%;
            display: flex;
            justify-content: space-between;
            padding: 0 20px;
            box-sizing: border-box;
            z-index: 10;
        }

        .control-group { display: flex; gap: 10px; }
        .btn {
            width: 75px;
            height: 75px;
            background: rgba(255, 255, 255, 0.7);
            border: 2px solid #fff;
            border-radius: 15px;
            display: flex;
            justify-content: center;
            align-items: center;
            font-weight: bold;
            user-select: none;
            backdrop-filter: blur(5px);
        }
        .btn:active { background: #fff; transform: scale(0.95); }

        /* Overlays */
        .overlay {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.8);
            display: none;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            color: white;
            z-index: 100;
            text-align: center;
        }

        .question-box {
            background: white;
            color: #333;
            padding: 20px;
            border-radius: 15px;
            width: 80%;
            max-width: 400px;
        }

        .option-btn {
            display: block;
            width: 100%;
            padding: 12px;
            margin: 10px 0;
            background: var(--accent);
            color: white;
            border: none;
            border-radius: 8px;
            font-size: 16px;
        }
    </style>
</head>
<body>

<div id="game-container">
    <div id="top-ui">
        <div class="ui-item"><span class="ui-label">SCORE</span><span id="score-val">0</span></div>
        <div class="ui-item"><span class="ui-label">SPEED</span><span id="speed-val">0</span> km/h</div>
        <div class="ui-item"><span class="ui-label">MISTAKES</span><span id="mistake-val">0/3</span></div>
        <div id="mini-signal" style="width:15px; height:15px; border-radius:50%; background:#777;"></div>
    </div>

    <canvas id="gameCanvas"></canvas>

    <div id="controls">
        <div class="control-group">
            <div class="btn" id="btn-left">LEFT</div>
            <div class="btn" id="btn-right">RIGHT</div>
        </div>
        <div class="control-group">
            <div class="btn" id="btn-brake" style="background: rgba(231, 76, 60, 0.5);">BRAKE</div>
            <div class="btn" id="btn-gas" style="background: rgba(46, 204, 113, 0.5);">GAS</div>
        </div>
    </div>

    <div id="question-overlay" class="overlay">
        <div class="question-box">
            <h3 id="question-text">Question?</h3>
            <div id="options-container"></div>
        </div>
    </div>

    <div id="game-over-overlay" class="overlay">
        <h1>GAME OVER</h1>
        <p>Your Final Score: <span id="final-score">0</span></p>
        <button class="option-btn" onclick="location.reload()">RESTART</button>
    </div>
</div>

<script>
/** * GAME CONFIGURATION 
 */
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const laneWidth = 80;
const roadWidth = laneWidth * 3;
const speedLimit = 60;

let gameState = {
    score: 0,
    speed: 0,
    mistakes: 0,
    isPaused: false,
    roadOffset: 0,
    carX: 0,
    carY: 0,
    targetSpeed: 0,
    objects: [],
    lastObjectSpawn: 0,
    mistakeCooldown: false
};

const questions = [
    { q: "What should you do at a solid red light?", a: ["Stop and wait", "Slow down", "Speed up"], correct: 0 },
    { q: "When a pedestrian is on a zebra crossing, you must:", a: ["Honk", "Stop completely", "Drive around them"], correct: 1 },
    { q: "A yellow signal means:", a: ["Speed up", "Prepare to stop", "Go faster"], correct: 1 }
];

// Input handling
const keys = { left: false, right: false, gas: false, brake: false };

/**
 * INITIALIZATION
 */
function init() {
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
    gameState.carX = canvas.width / 2;
    gameState.carY = canvas.height - 150;
    update();
}

/**
 * GAME LOOP
 */
function update() {
    if (gameState.isPaused) return;

    // Movement Logic
    if (keys.gas) gameState.targetSpeed = 80;
    else if (keys.brake) gameState.targetSpeed = 0;
    else gameState.targetSpeed = 30; // Coasting speed

    // Smooth speed transition
    gameState.speed += (gameState.targetSpeed - gameState.speed) * 0.05;
    if (gameState.speed < 0.5) gameState.speed = 0;

    // Steering
    if (keys.left) gameState.carX -= 4;
    if (keys.right) gameState.carX += 4;

    // Road constraints
    const leftBound = (canvas.width - roadWidth) / 2;
    const rightBound = (canvas.width + roadWidth) / 2;
    gameState.carX = Math.max(leftBound + 20, Math.min(rightBound - 20, gameState.carX));

    // Update UI
    document.getElementById('score-val').innerText = Math.floor(gameState.score);
    document.getElementById('speed-val').innerText = Math.floor(gameState.speed);
    document.getElementById('mistake-val').innerText = `${gameState.mistakes}/3`;

    // Overspeed check
    if (gameState.speed > speedLimit + 5) {
        triggerMistake("Speed Limit Exceeded!");
    }

    // Object Spawning
    if (Date.now() - gameState.lastObjectSpawn > 3000) {
        spawnObject();
        gameState.lastObjectSpawn = Date.now();
    }

    // Logic for objects
    updateObjects();

    draw();
    requestAnimationFrame(update);
}

/**
 * OBJECTS SYSTEM (Lights & Pedestrians)
 */
function spawnObject() {
    const type = Math.random() > 0.5 ? 'LIGHT' : 'PEDESTRIAN';
    const obj = {
        type: type,
        y: -100,
        x: canvas.width / 2,
        passed: false,
        triggered: false
    };

    if (type === 'LIGHT') {
        obj.state = 'GREEN';
        obj.timer = 0;
    } else {
        obj.pedX = (canvas.width - roadWidth) / 2 - 30;
        obj.pedCrossing = false;
    }
    gameState.objects.push(obj);
}

function updateObjects() {
    gameState.objects.forEach(obj => {
        obj.y += gameState.speed * 0.1;

        if (obj.type === 'LIGHT') {
            obj.timer++;
            if (obj.timer < 150) { obj.state = 'GREEN'; document.getElementById('mini-signal').style.background = '#2ecc71'; }
            else if (obj.timer < 250) { obj.state = 'YELLOW'; document.getElementById('mini-signal').style.background = '#f1c40f'; }
            else if (obj.timer < 450) { obj.state = 'RED'; document.getElementById('mini-signal').style.background = '#e74c3c'; }
            else obj.timer = 0;

            // Rule Check: Red Light
            if (obj.state === 'RED' && !obj.triggered && obj.y > gameState.carY - 50 && obj.y < gameState.carY) {
                if (gameState.speed > 5) {
                    triggerMistake("Ran a Red Light!");
                    obj.triggered = true;
                }
            }
        }

        if (obj.type === 'PEDESTRIAN') {
            // Logic to move pedestrian across
            if (obj.y > 100 && obj.y < canvas.height) {
                obj.pedX += 2;
            }
            // Collision or Failure to Stop
            const dist = Math.abs(obj.y - gameState.carY);
            if (dist < 100 && obj.pedX > (canvas.width - roadWidth)/2 && obj.pedX < (canvas.width + roadWidth)/2) {
                if (gameState.speed > 2) {
                    triggerMistake("Failed to yield to pedestrian!");
                }
            }
            // Hit check
            if (dist < 20 && Math.abs(gameState.carX - obj.pedX) < 30) {
                gameOver();
            }
        }

        // Score for passing correctly
        if (obj.y > gameState.carY + 50 && !obj.passed) {
            gameState.score += 10;
            obj.passed = true;
        }
    });

    // Cleanup
    gameState.objects = gameState.objects.filter(o => o.y < canvas.height + 100);
}

/**
 * MISTAKE & QUESTION SYSTEM
 */
function triggerMistake(msg) {
    if (gameState.mistakeCooldown) return;
    gameState.mistakes++;
    gameState.mistakeCooldown = true;
    setTimeout(() => gameState.mistakeCooldown = false, 2000);

    if (gameState.mistakes >= 3) {
        showQuestion();
    }
}

function showQuestion() {
    gameState.isPaused = true;
    const q = questions[Math.floor(Math.random() * questions.length)];
    const overlay = document.getElementById('question-overlay');
    const container = document.getElementById('options-container');
    
    document.getElementById('question-text').innerText = q.q;
    container.innerHTML = '';
    overlay.style.display = 'flex';

    q.a.forEach((opt, index) => {
        const btn = document.createElement('button');
        btn.className = 'option-btn';
        btn.innerText = opt;
        btn.onclick = () => {
            if (index === q.correct) {
                gameState.mistakes = 0;
                gameState.isPaused = false;
                overlay.style.display = 'none';
                update();
            } else {
                gameOver();
            }
        };
        container.appendChild(btn);
    });
}

function gameOver() {
    gameState.isPaused = true;
    document.getElementById('game-over-overlay').style.display = 'flex';
    document.getElementById('final-score').innerText = Math.floor(gameState.score);
}

/**
 * DRAWING
 */
function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    const centerX = canvas.width / 2;

    // Road
    ctx.fillStyle = '#444';
    ctx.fillRect(centerX - roadWidth/2, 0, roadWidth, canvas.height);

    // Lane Lines
    ctx.strokeStyle = '#fff';
    ctx.setLineDash([20, 20]);
    ctx.lineWidth = 4;
    ctx.beginPath();
    ctx.moveTo(centerX - laneWidth/2, 0); ctx.lineTo(centerX - laneWidth/2, canvas.height);
    ctx.moveTo(centerX + laneWidth/2, 0); ctx.lineTo(centerX + laneWidth/2, canvas.height);
    ctx.stroke();

    // Objects
    gameState.objects.forEach(obj => {
        if (obj.type === 'LIGHT') {
            ctx.fillStyle = '#222';
            ctx.fillRect(centerX + roadWidth/2 + 10, obj.y - 40, 30, 80);
            ctx.fillStyle = (obj.state === 'RED') ? '#ff0000' : '#330000';
            ctx.beginPath(); ctx.arc(centerX + roadWidth/2 + 25, obj.y - 25, 8, 0, Math.PI*2); ctx.fill();
            ctx.fillStyle = (obj.state === 'YELLOW') ? '#ffff00' : '#333300';
            ctx.beginPath(); ctx.arc(centerX + roadWidth/2 + 25, obj.y, 8, 0, Math.PI*2); ctx.fill();
            ctx.fillStyle = (obj.state === 'GREEN') ? '#00ff00' : '#003300';
            ctx.beginPath(); ctx.arc(centerX + roadWidth/2 + 25, obj.y + 25, 8, 0, Math.PI*2); ctx.fill();
            
            // Stop Line
            ctx.strokeStyle = 'white';
            ctx.setLineDash([]);
            ctx.strokeRect(centerX - roadWidth/2, obj.y + 50, roadWidth, 5);
        }

        if (obj.type === 'PEDESTRIAN') {
            // Zebra Crossing
            ctx.fillStyle = 'rgba(255,255,255,0.8)';
            for(let i=0; i<6; i++) {
                ctx.fillRect(centerX - roadWidth/2 + (i*40), obj.y - 20, 30, 100);
            }
            // Pedestrian
            ctx.fillStyle = '#e67e22';
            ctx.beginPath(); ctx.arc(obj.pedX, obj.y + 30, 10, 0, Math.PI*2); ctx.fill();
        }
    });

    // Player Car
    ctx.fillStyle = '#3498db';
    ctx.save();
    ctx.translate(gameState.carX, gameState.carY);
    ctx.fillRect(-15, -25, 30, 50); // Body
    ctx.fillStyle = '#222';
    ctx.fillRect(-18, -20, 5, 10); // Wheels
    ctx.fillRect(13, -20, 5, 10);
    ctx.restore();
}

/**
 * CONTROLS BINDING
 */
function bindEvents() {
    const setupBtn = (id, key) => {
        const el = document.getElementById(id);
        el.addEventListener('touchstart', (e) => { e.preventDefault(); keys[key] = true; });
        el.addEventListener('touchend', (e) => { e.preventDefault(); keys[key] = false; });
        el.addEventListener('mousedown', () => keys[key] = true);
        el.addEventListener('mouseup', () => keys[key] = false);
    };

    setupBtn('btn-left', 'left');
    setupBtn('btn-right', 'right');
    setupBtn('btn-gas', 'gas');
    setupBtn('btn-brake', 'brake');

    window.addEventListener('keydown', (e) => {
        if (e.key === 'ArrowLeft') keys.left = true;
        if (e.key === 'ArrowRight') keys.right = true;
        if (e.key === 'ArrowUp') keys.gas = true;
        if (e.key === 'ArrowDown') keys.brake = true;
    });

    window.addEventListener('keyup', (e) => {
        if (e.key === 'ArrowLeft') keys.left = false;
        if (e.key === 'ArrowRight') keys.right = false;
        if (e.key === 'ArrowUp') keys.gas = false;
        if (e.key === 'ArrowDown') keys.brake = false;
    });
}

window.onload = () => {
    bindEvents();
    init();
};
</script>
</body>
</html>
