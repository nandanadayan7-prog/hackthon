
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Traffic Rule Racer</title>
    <style>
        :root {
            --bg-color: #2c3e50;
            --road-color: #34495e;
            --ui-panel: rgba(255, 255, 255, 0.95);
            --accent: #3498db;
            --danger: #e74c3c;
            --success: #2ecc71;
        }

        body {
            margin: 0;
            padding: 0;
            overflow: hidden;
            font-family: 'Segoe UI', system-ui, sans-serif;
            background: #27ae60; /* Grass color */
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
            top: 15px;
            width: 90%;
            max-width: 600px;
            height: 60px;
            background: var(--ui-panel);
            border-radius: 15px;
            display: flex;
            justify-content: space-around;
            align-items: center;
            box-shadow: 0 8px 32px rgba(0,0,0,0.2);
            z-index: 10;
            border-bottom: 3px solid #ddd;
        }

        .ui-item { text-align: center; font-weight: 800; font-size: 16px; color: #2c3e50; }
        .ui-label { font-size: 10px; color: #7f8c8d; display: block; letter-spacing: 1px; }

        #speedometer-ring {
            width: 12px; height: 12px; border-radius: 50%; display: inline-block; margin-right: 5px;
        }

        /* Controls */
        #controls {
            position: absolute;
            bottom: 40px;
            width: 100%;
            display: flex;
            justify-content: space-between;
            padding: 0 25px;
            box-sizing: border-box;
            z-index: 10;
        }

        .control-group { display: flex; gap: 15px; }
        .btn {
            width: 85px;
            height: 85px;
            background: rgba(255, 255, 255, 0.2);
            border: 3px solid #fff;
            border-radius: 20px;
            display: flex;
            justify-content: center;
            align-items: center;
            color: white;
            font-weight: bold;
            font-size: 12px;
            backdrop-filter: blur(10px);
            user-select: none;
            transition: transform 0.1s, background 0.1s;
        }
        .btn:active { transform: scale(0.9); background: rgba(255, 255, 255, 0.5); }

        /* Overlays */
        .overlay {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(44, 62, 80, 0.9);
            display: none;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            color: white;
            z-index: 100;
        }

        .question-box {
            background: white;
            color: #333;
            padding: 30px;
            border-radius: 20px;
            width: 85%;
            max-width: 400px;
            box-shadow: 0 10px 40px rgba(0,0,0,0.5);
        }

        .option-btn {
            display: block;
            width: 100%;
            padding: 15px;
            margin: 12px 0;
            background: var(--accent);
            color: white;
            border: none;
            border-radius: 10px;
            font-size: 16px;
            font-weight: bold;
            cursor: pointer;
        }

        canvas { display: block; }
    </style>
</head>
<body>

<div id="game-container">
    <div id="top-ui">
        <div class="ui-item"><span class="ui-label">SCORE</span><span id="score-val">0</span></div>
        <div class="ui-item">
            <span class="ui-label">SPEED</span>
            <div id="speedometer-ring" style="background: var(--success);"></div>
            <span id="speed-val">0</span> <small>km/h</small>
        </div>
        <div class="ui-item"><span class="ui-label">MISTAKES</span><span id="mistake-val">0 / 3</span></div>
    </div>

    <canvas id="gameCanvas"></canvas>

    <div id="controls">
        <div class="control-group">
            <div class="btn" id="btn-left">LEFT</div>
            <div class="btn" id="btn-right">RIGHT</div>
        </div>
        <div class="control-group">
            <div class="btn" id="btn-brake" style="border-color: var(--danger); color: var(--danger);">BRAKE</div>
            <div class="btn" id="btn-gas" style="border-color: var(--success); color: var(--success);">GAS</div>
        </div>
    </div>

    <div id="question-overlay" class="overlay">
        <div class="question-box">
            <h2 style="margin-top:0">Traffic Review</h2>
            <p id="question-text" style="font-size: 18px; line-height: 1.4;">Question goes here?</p>
            <div id="options-container"></div>
        </div>
    </div>

    <div id="game-over-overlay" class="overlay">
        <h1 style="font-size: 48px; margin-bottom: 0;">GAME OVER</h1>
        <p style="font-size: 20px; opacity: 0.8;">Safety First!</p>
        <div style="background: white; color: #333; padding: 20px; border-radius: 15px; margin: 20px; width: 200px;">
            <span class="ui-label">FINAL SCORE</span>
            <h2 id="final-score" style="margin: 5px 0;">0</h2>
        </div>
        <button class="option-btn" style="width: 200px;" onclick="location.reload()">TRY AGAIN</button>
    </div>
</div>

<script>
/** 🏎️ GAME CORE CONFIG **/
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const laneWidth = 90;
const roadWidth = laneWidth * 3;
const speedLimit = 60;
const minGapBetweenEvents = 900; // Increased gap for better pacing

let gameState = {
    score: 0,
    speed: 0,
    mistakes: 0,
    isPaused: false,
    distanceTraveled: 0,
    lastSpawnDistance: 0,
    carX: 0,
    carY: 0,
    targetSpeed: 0,
    objects: [],
    mistakeCooldown: false
};

const questions = [
    { q: "You approach a yellow light. What is the safest action?", a: ["Speed up to clear it", "Slow down and prepare to stop", "Maintain speed"], correct: 1 },
    { q: "A pedestrian is waiting at a zebra crossing. You must:", a: ["Honk to warn them", "Slow down and stop", "Swerve to the left lane"], correct: 1 },
    { q: "What is the primary purpose of a speed limit?", a: ["To collect fines", "To ensure road safety", "To test your engine"], correct: 1 }
];

const keys = { left: false, right: false, gas: false, brake: false };

function init() {
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
    gameState.carX = canvas.width / 2;
    gameState.carY = canvas.height - 180;
    update();
}

/** ⚙️ LOGIC UPDATES **/
function update() {
    if (gameState.isPaused) return;

    // 1. Movement Logic
    if (keys.gas) gameState.targetSpeed = 90; 
    else if (keys.brake) gameState.targetSpeed = 0;
    else gameState.targetSpeed = 40; // Default cruise

    gameState.speed += (gameState.targetSpeed - gameState.speed) * 0.04;
    if (gameState.speed < 0.5) gameState.speed = 0;

    // 2. Distance & Spawning
    gameState.distanceTraveled += gameState.speed * 0.1;
    
    // Check if it's time to spawn a new rule event
    if (gameState.distanceTraveled - gameState.lastSpawnDistance > minGapBetweenEvents) {
        spawnObject();
        gameState.lastSpawnDistance = gameState.distanceTraveled;
    }

    // 3. UI Updates
    document.getElementById('score-val').innerText = Math.floor(gameState.score);
    document.getElementById('speed-val').innerText = Math.floor(gameState.speed);
    document.getElementById('mistake-val').innerText = `${gameState.mistakes} / 3`;
    
    const speedRing = document.getElementById('speedometer-ring');
    speedRing.style.background = gameState.speed > speedLimit ? 'var(--danger)' : 'var(--success)';

    // 4. Object Handling
    handleObjects();

    // 5. Steering
    if (keys.left) gameState.carX -= 5;
    if (keys.right) gameState.carX += 5;
    const roadLeft = (canvas.width - roadWidth) / 2;
    const roadRight = (canvas.width + roadWidth) / 2;
    gameState.carX = Math.max(roadLeft + 25, Math.min(roadRight - 25, gameState.carX));

    draw();
    requestAnimationFrame(update);
}

function spawnObject() {
    const type = Math.random() > 0.5 ? 'LIGHT' : 'PEDESTRIAN';
    const obj = {
        type: type,
        y: -200,
        passed: false,
        triggered: false,
        mistakeHandled: false
    };

    if (type === 'LIGHT') {
        obj.timer = 0;
        obj.state = 'GREEN';
    } else {
        obj.pedX = (canvas.width - roadWidth) / 2 - 40;
        obj.isWalking = false;
    }
    gameState.objects.push(obj);
}

function handleObjects() {
    gameState.objects.forEach(obj => {
        obj.y += gameState.speed * 0.1; // Move object relative to speed

        if (obj.type === 'LIGHT') {
            obj.timer++;
            if (obj.timer < 180) obj.state = 'GREEN';
            else if (obj.timer < 300) obj.state = 'YELLOW';
            else if (obj.timer < 550) obj.state = 'RED';
            else obj.timer = 0;

            // Stop line logic (approx 50px below the light)
            const stopLineY = obj.y + 60;
            if (obj.state === 'RED' && !obj.mistakeHandled) {
                if (gameState.carY < stopLineY + 20 && gameState.carY > stopLineY - 20 && gameState.speed > 5) {
                    triggerMistake("Ran a Red Light!");
                    obj.mistakeHandled = true;
                }
            }
        }

        if (obj.type === 'PEDESTRIAN') {
            // Pedestrian starts crossing when car is close
            if (gameState.carY - obj.y < 450) obj.isWalking = true;
            if (obj.isWalking && obj.pedX < (canvas.width + roadWidth) / 2 + 50) {
                obj.pedX += 2;
            }

            // Failure to yield check
            const roadLeft = (canvas.width - roadWidth) / 2;
            const roadRight = (canvas.width + roadWidth) / 2;
            const isCrossing = obj.pedX > roadLeft && obj.pedX < roadRight;
            
            if (isCrossing && Math.abs(gameState.carY - obj.y) < 100 && gameState.speed > 5 && !obj.mistakeHandled) {
                triggerMistake("Yield to Pedestrians!");
                obj.mistakeHandled = true;
            }

            // Game Over Collision
            if (Math.abs(gameState.carY - (obj.y + 40)) < 30 && Math.abs(gameState.carX - obj.pedX) < 25) {
                gameOver();
            }
        }

        if (obj.y > gameState.carY + 100 && !obj.passed) {
            if (!obj.mistakeHandled) gameState.score += 20; // Perfect pass
            obj.passed = true;
        }
    });

    gameState.objects = gameState.objects.filter(o => o.y < canvas.height + 200);
}

/** ⚠️ MISTAKE SYSTEM **/
function triggerMistake(msg) {
    if (gameState.mistakeCooldown) return;
    gameState.mistakes++;
    gameState.mistakeCooldown = true;
    setTimeout(() => gameState.mistakeCooldown = false, 2000);

    if (gameState.mistakes >= 3) {
        gameState.isPaused = true;
        showQuestion();
    }
}

function showQuestion() {
    const q = questions[Math.floor(Math.random() * questions.length)];
    const overlay = document.getElementById('question-overlay');
    const container = document.getElementById('options-container');
    
    document.getElementById('question-text').innerText = q.q;
    container.innerHTML = '';
    overlay.style.display = 'flex';

    q.a.forEach((opt, idx) => {
        const btn = document.createElement('button');
        btn.className = 'option-btn';
        btn.innerText = opt;
        btn.onclick = () => {
            if (idx === q.correct) {
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

/** 🎨 DRAWING ENGINE **/
function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    const centerX = canvas.width / 2;

    // Grass & Decor
    ctx.fillStyle = '#27ae60';
    ctx.fillRect(0,0, canvas.width, canvas.height);

    // Road
    ctx.fillStyle = '#34495e';
    ctx.fillRect(centerX - roadWidth/2, 0, roadWidth, canvas.height);

    // Side markings
    ctx.fillStyle = '#f1c40f';
    ctx.fillRect(centerX - roadWidth/2 - 5, 0, 5, canvas.height);
    ctx.fillRect(centerX + roadWidth/2, 0, 5, canvas.height);

    // Lane Dashes
    ctx.strokeStyle = 'rgba(255,255,255,0.3)';
    ctx.setLineDash([30, 40]);
    ctx.lineWidth = 4;
    ctx.beginPath();
    ctx.moveTo(centerX - laneWidth/2, 0); ctx.lineTo(centerX - laneWidth/2, canvas.height);
    ctx.moveTo(centerX + laneWidth/2, 0); ctx.lineTo(centerX + laneWidth/2, canvas.height);
    ctx.stroke();

    gameState.objects.forEach(obj => {
        if (obj.type === 'LIGHT') {
            // Post
            ctx.fillStyle = '#222';
            ctx.fillRect(centerX + roadWidth/2 + 10, obj.y - 40, 30, 90);
            // Bulbs
            ctx.fillStyle = (obj.state === 'RED') ? '#ff4757' : '#331111';
            ctx.beginPath(); ctx.arc(centerX + roadWidth/2 + 25, obj.y - 25, 10, 0, 7); ctx.fill();
            ctx.fillStyle = (obj.state === 'YELLOW') ? '#eccc68' : '#333311';
            ctx.beginPath(); ctx.arc(centerX + roadWidth/2 + 25, obj.y + 5, 10, 0, 7); ctx.fill();
            ctx.fillStyle = (obj.state === 'GREEN') ? '#2ed573' : '#113311';
            ctx.beginPath(); ctx.arc(centerX + roadWidth/2 + 25, obj.y + 35, 10, 0, 7); ctx.fill();
            
            // Stop line
            ctx.fillStyle = 'white';
            ctx.fillRect(centerX - roadWidth/2, obj.y + 60, roadWidth, 8);
        }

        if (obj.type === 'PEDESTRIAN') {
            // Zebra strips
            ctx.fillStyle = 'rgba(255,255,255,0.7)';
            for(let i=0; i<8; i++) {
                ctx.fillRect(centerX - roadWidth/2 + (i*40), obj.y, 25, 80);
            }
            // Pedestrian person
            ctx.fillStyle = '#e67e22';
            ctx.beginPath(); ctx.arc(obj.pedX, obj.y + 40, 12, 0, 7); ctx.fill();
        }
    });

    // Player Car
    ctx.save();
    ctx.translate(gameState.carX, gameState.carY);
    // Shadow
    ctx.fillStyle = 'rgba(0,0,0,0.2)';
    ctx.fillRect(-18, -22, 40, 55);
    // Body
    ctx.fillStyle = '#3498db';
    ctx.fillRect(-20, -30, 40, 60);
    // Windshield
    ctx.fillStyle = '#ecf0f1';
    ctx.fillRect(-15, -20, 30, 15);
    // Tail lights
    ctx.fillStyle = (keys.brake) ? '#ff0000' : '#880000';
    ctx.fillRect(-18, 25, 10, 5);
    ctx.fillRect(8, 25, 10, 5);
    ctx.restore();
}

/** 🎮 EVENT BINDING **/
const setupBtn = (id, key) => {
    const el = document.getElementById(id);
    const start = (e) => { e.preventDefault(); keys[key] = true; };
    const end = (e) => { e.preventDefault(); keys[key] = false; };
    el.addEventListener('touchstart', start);
    el.addEventListener('touchend', end);
    el.addEventListener('mousedown', start);
    el.addEventListener('mouseup', end);
    el.addEventListener('mouseleave', end);
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

window.onload = init;
</script>
</body>
</html>
