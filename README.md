<!DOCTYPE html>
<html lang="bn">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Samba Runner BD ⚽</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; touch-action: manipulation; }
        html, body {
            width: 100%;
            height: 100%;
            overflow: hidden;
            background: #0a0f1d;
            font-family: 'Poppins', 'Segoe UI', sans-serif;
            user-select: none;
            -webkit-user-select: none;
        }

        #game-wrapper {
            position: relative;
            width: 100vw;
            height: 100vh;
            overflow: hidden;
            background: #000;
        }

        canvas { display: block; width: 100%; height: 100%; }

        /* UI Screens */
        .overlay {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(10, 15, 29, 0.92);
            backdrop-filter: blur(8px);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 10;
            text-align: center;
            padding: 15px;
        }

        .title-container {
            margin-bottom: 15px;
            animation: bounce 2s infinite ease-in-out;
        }

        h1.game-title {
            font-size: clamp(24px, 6vw, 48px);
            font-weight: 900;
            background: linear-gradient(45deg, #fed100, #009c3b, #002776);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            text-shadow: 0 5px 15px rgba(0,0,0,0.5);
            letter-spacing: 1px;
        }

        .sub-title {
            font-size: clamp(12px, 3vw, 18px);
            color: #ffffff;
            margin-top: 4px;
            font-weight: 800;
            letter-spacing: 2px;
            text-transform: uppercase;
        }

        p.instructions {
            font-size: clamp(14px, 3.5vw, 22px);
            font-weight: 800;
            margin-bottom: 20px;
            color: #ffffff;
            background: rgba(255, 255, 255, 0.12);
            padding: 8px 20px;
            border-radius: 30px;
            border: 2px solid rgba(254, 209, 0, 0.6);
        }

        .btn {
            padding: 12px 30px;
            font-size: clamp(14px, 3vw, 18px);
            font-weight: 800;
            color: #fff;
            background: linear-gradient(135deg, #009c3b 0%, #006837 100%);
            border: 2px solid #fed100;
            border-radius: 50px;
            cursor: pointer;
            outline: none;
            margin: 5px;
            text-transform: uppercase;
        }

        .btn:active {
            transform: scale(0.95);
        }

        #hud {
            position: absolute;
            top: 10px; left: 10px; right: 10px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            z-index: 5;
            font-size: clamp(12px, 2.8vw, 16px);
            font-weight: 700;
            color: #fff;
            pointer-events: none;
        }

        .hud-badge {
            background: rgba(0, 0, 0, 0.7);
            padding: 6px 12px;
            border-radius: 20px;
            border: 1px solid rgba(254, 209, 0, 0.4);
            display: inline-flex;
            align-items: center;
            gap: 5px;
        }

        .hud-btn {
            pointer-events: auto;
            padding: 6px 14px;
            font-size: clamp(10px, 2.5vw, 14px);
            border-radius: 20px;
        }

        .hidden { display: none !important; }

        @keyframes bounce {
            0%, 100% { transform: translateY(0); }
            50% { transform: translateY(-5px); }
        }
    </style>
</head>
<body>

<div id="game-wrapper">
    <audio id="bgMusic" src="7upbg.mp3" loop preload="auto"></audio>

    <div id="hud">
        <div style="display: flex; gap: 6px;">
            <div class="hud-badge">⚽ <span id="scoreText" style="color: #fed100;">0</span></div>
            <div class="hud-badge">🪙 <span id="coinText" style="color: #ffd700;">0</span></div>
            <div class="hud-badge">🏆 <span id="highScoreText" style="color: #00ff87;">0</span></div>
        </div>
        <div>
            <button id="pauseBtn" class="btn hud-btn">Pause ⏸️</button>
        </div>
    </div>

    <canvas id="gameCanvas"></canvas>

    <!-- Start Screen -->
    <div id="startScreen" class="overlay">
        <div class="title-container">
            <h1 class="game-title">Samba Runner BD ⚽</h1>
            <div class="sub-title">RRX STUDIOS PRESENTS</div>
        </div>
        <p class="instructions">7 আপ খাও, হেক্সা মিশন জিতো!</p>
        <button id="startBtn" class="btn">START GAME ▶</button>
    </div>

    <!-- Pause Screen -->
    <div id="pauseScreen" class="overlay hidden">
        <h1 class="game-title">GAME PAUSED ⏸️</h1>
        <br>
        <button id="resumeBtn" class="btn">RESUME ▶</button>
        <button id="menuBtn1" class="btn">MAIN MENU 🏠</button>
    </div>

    <!-- Game Over Screen -->
    <div id="gameOverScreen" class="overlay hidden">
        <h1 style="color: #ff4757; font-size: clamp(28px, 7vw, 42px);">ELIMINATED! 💥</h1>
        <p style="font-size: clamp(14px, 3.5vw, 20px); margin: 10px 0;">
            Score: <span id="finalScore" style="color: #fed100;">0</span> | 
            Coins: <span id="finalCoins" style="color: #ffd700;">0</span>
        </p>
        <div>
            <button id="restartBtn" class="btn">PLAY AGAIN 🔄</button>
            <button id="menuBtn2" class="btn">MAIN MENU 🏠</button>
        </div>
    </div>
</div>

<script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const gameWrapper = document.getElementById('game-wrapper');

    const startScreen = document.getElementById('startScreen');
    const pauseScreen = document.getElementById('pauseScreen');
    const gameOverScreen = document.getElementById('gameOverScreen');
    const startBtn = document.getElementById('startBtn');
    const pauseBtn = document.getElementById('pauseBtn');
    const resumeBtn = document.getElementById('resumeBtn');
    const restartBtn = document.getElementById('restartBtn');
    const menuBtn1 = document.getElementById('menuBtn1');
    const menuBtn2 = document.getElementById('menuBtn2');
    
    const scoreText = document.getElementById('scoreText');
    const coinText = document.getElementById('coinText');
    const highScoreText = document.getElementById('highScoreText');
    const finalScore = document.getElementById('finalScore');
    const finalCoins = document.getElementById('finalCoins');
    const bgMusic = document.getElementById('bgMusic');

    let highScore = localStorage.getItem('samba_runner_highscore') || 0;
    highScoreText.innerText = highScore;

    // Dynamic Screen Resize Logic
    let groundY = 0;
    function resizeCanvas() {
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;
        groundY = canvas.height * 0.8;
    }
    window.addEventListener('resize', resizeCanvas);
    resizeCanvas();

    const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    function playSound(type) {
        if (audioCtx.state === 'suspended') audioCtx.resume();
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        osc.connect(gain);
        gain.connect(audioCtx.destination);
        const now = audioCtx.currentTime;

        if (type === 'jump') {
            osc.type = 'sine';
            osc.frequency.setValueAtTime(180, now);
            osc.frequency.exponentialRampToValueAtTime(520, now + 0.15);
            gain.gain.setValueAtTime(0.3, now);
            gain.gain.linearRampToValueAtTime(0.01, now + 0.15);
            osc.start(now);
            osc.stop(now + 0.15);
        } else if (type === 'coin') {
            osc.type = 'triangle';
            osc.frequency.setValueAtTime(600, now);
            osc.frequency.setValueAtTime(900, now + 0.08);
            gain.gain.setValueAtTime(0.25, now);
            gain.gain.linearRampToValueAtTime(0.01, now + 0.2);
            osc.start(now);
            osc.stop(now + 0.2);
        } else if (type === 'hit') {
            osc.type = 'sawtooth';
            osc.frequency.setValueAtTime(150, now);
            osc.frequency.exponentialRampToValueAtTime(40, now + 0.3);
            gain.gain.setValueAtTime(0.4, now);
            gain.gain.linearRampToValueAtTime(0.01, now + 0.3);
            osc.start(now);
            osc.stop(now + 0.3);
        }
    }

    function playBGM() { if (bgMusic) bgMusic.play().catch(e => {}); }
    function pauseBGM() { if (bgMusic) bgMusic.pause(); }
    function stopBGM() { if (bgMusic) { bgMusic.pause(); bgMusic.currentTime = 0; } }

    let gameState = 'START';
    let score = 0;
    let coinsCollected = 0;
    let gameSpeed = 6;
    let frameCount = 0;
    const gravity = 0.65;

    const player = {
        x: 50,
        y: 0,
        width: 36,
        height: 56,
        dy: 0,
        isJumping: false,
        
        draw() {
            ctx.save();
            ctx.translate(this.x + this.width / 2, this.y + this.height / 2);
            let legAngle = !this.isJumping ? Math.sin(frameCount * 0.25) * 0.6 : 0.4;

            ctx.strokeStyle = '#002776';
            ctx.lineWidth = 6;
            ctx.beginPath();
            ctx.moveTo(-4, 8);
            ctx.lineTo(-Math.sin(legAngle) * 18, 24);
            ctx.stroke();

            ctx.beginPath();
            ctx.moveTo(4, 8);
            ctx.lineTo(Math.sin(legAngle) * 18, 24);
            ctx.stroke();

            ctx.fillStyle = '#fed100';
            ctx.beginPath();
            ctx.roundRect(-13, -16, 26, 26, 4);
            ctx.fill();

            ctx.fillStyle = '#009c3b';
            ctx.fillRect(-13, -16, 26, 5);

            ctx.fillStyle = '#002776';
            ctx.font = 'bold 10px sans-serif';
            ctx.textAlign = 'center';
            ctx.fillText('10', 0, 0);

            ctx.fillStyle = '#e0ac69';
            ctx.beginPath();
            ctx.arc(0, -25, 10, 0, Math.PI * 2);
            ctx.fill();

            ctx.fillStyle = '#222';
            ctx.beginPath();
            ctx.arc(0, -28, 10, Math.PI * 0.8, Math.PI * 2.2);
            ctx.fill();

            ctx.restore();
        },
        
        jump() {
            if (!this.isJumping) {
                this.dy = -13.5;
                this.isJumping = true;
                playSound('jump');
            }
        },

        update() {
            this.dy += gravity;
            this.y += this.dy;

            if (this.y + this.height >= groundY) {
                this.y = groundY - this.height;
                this.dy = 0;
                this.isJumping = false;
            }
            this.draw();
        }
    };

    let obstacles = [];
    class Obstacle {
        constructor() {
            this.type = Math.random() > 0.4 ? '7UP' : 'SPIKE';
            this.x = canvas.width + 30;
            this.width = this.type === '7UP' ? 30 : 34;
            this.height = this.type === '7UP' ? 50 : 35;
            this.y = groundY - this.height;
        }

        draw() {
            if (this.type === '7UP') {
                ctx.fillStyle = '#00833e';
                ctx.beginPath();
                ctx.roundRect(this.x, this.y, this.width, this.height, 4);
                ctx.fill();
                ctx.fillStyle = '#ffffff';
                ctx.font = 'bold 11px sans-serif';
                ctx.fillText('7 Up', this.x + 3, this.y + 26);
            } else {
                ctx.fillStyle = '#d63031';
                ctx.beginPath();
                ctx.moveTo(this.x, groundY);
                ctx.lineTo(this.x + this.width/2, groundY - this.height);
                ctx.lineTo(this.x + this.width, groundY);
                ctx.closePath();
                ctx.fill();
            }
        }

        update() {
            this.x -= gameSpeed;
            this.draw();
        }
    }

    let coins = [];
    class Coin {
        constructor() {
            this.x = canvas.width + 20;
            this.y = groundY - 40 - Math.random() * 50;
            this.radius = 12;
        }

        draw() {
            ctx.fillStyle = '#ffd700';
            ctx.beginPath();
            ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
            ctx.fill();
            ctx.fillStyle = '#b8860b';
            ctx.font = 'bold 10px sans-serif';
            ctx.textAlign = 'center';
            ctx.textBaseline = 'middle';
            ctx.fillText('⚽', this.x, this.y);
        }

        update() {
            this.x -= gameSpeed;
            this.draw();
        }
    }

    function drawBackground() {
        ctx.fillStyle = '#0f1f38';
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        ctx.fillStyle = '#111827';
        ctx.fillRect(0, groundY - 80, canvas.width, 80);

        ctx.fillStyle = '#1e7e34';
        ctx.fillRect(0, groundY, canvas.width, canvas.height - groundY);
        ctx.fillStyle = '#ffffff';
        ctx.fillRect(0, groundY, canvas.width, 3);
    }

    function handleSpawns() {
        if (frameCount % 100 === 0) obstacles.push(new Obstacle());
        if (frameCount % 130 === 0) coins.push(new Coin());
    }

    function checkCollisions() {
        for (let obs of obstacles) {
            if (
                player.x + 5 < obs.x + obs.width &&
                player.x + player.width - 5 > obs.x &&
                player.y + 5 < obs.y + obs.height &&
                player.y + player.height > obs.y
            ) {
                playSound('hit');
                gameOver();
            }
        }

        for (let i = coins.length - 1; i >= 0; i--) {
            let coin = coins[i];
            let dx = (player.x + player.width/2) - coin.x;
            let dy = (player.y + player.height/2) - coin.y;
            if (Math.sqrt(dx * dx + dy * dy) < player.width/2 + coin.radius) {
                coinsCollected++;
                coinText.innerText = coinsCollected;
                playSound('coin');
                coins.splice(i, 1);
            }
        }
    }

    function animate() {
        if (gameState !== 'PLAYING') return;

        ctx.clearRect(0, 0, canvas.width, canvas.height);
        frameCount++;

        drawBackground();
        handleSpawns();

        for (let i = obstacles.length - 1; i >= 0; i--) {
            obstacles[i].update();
            if (obstacles[i].x < -40) obstacles.splice(i, 1);
        }

        for (let i = coins.length - 1; i >= 0; i--) {
            coins[i].update();
            if (coins[i].x < -30) coins.splice(i, 1);
        }

        player.update();
        checkCollisions();

        if (frameCount % 6 === 0) {
            score++;
            scoreText.innerText = score;
        }

        if (frameCount % 450 === 0) gameSpeed += 0.3;

        requestAnimationFrame(animate);
    }

    function startGame() {
        resetData();
        gameState = 'PLAYING';
        startScreen.classList.add('hidden');
        pauseScreen.classList.add('hidden');
        gameOverScreen.classList.add('hidden');
        playBGM();
        animate();
    }

    function pauseGame() {
        if (gameState === 'PLAYING') {
            gameState = 'PAUSED';
            pauseBGM();
            pauseScreen.classList.remove('hidden');
        }
    }

    function resumeGame() {
        gameState = 'PLAYING';
        pauseScreen.classList.add('hidden');
        playBGM();
        animate();
    }

    function gameOver() {
        gameState = 'GAMEOVER';
        pauseBGM();

        if (score > highScore) {
            highScore = score;
            localStorage.setItem('samba_runner_highscore', highScore);
            highScoreText.innerText = highScore;
        }

        finalScore.innerText = score;
        finalCoins.innerText = coinsCollected;
        gameOverScreen.classList.remove('hidden');
    }

    function showMenu() {
        gameState = 'START';
        stopBGM();
        resetData();
        startScreen.classList.remove('hidden');
        pauseScreen.classList.add('hidden');
        gameOverScreen.classList.add('hidden');
        drawBackground();
    }

    function resetData() {
        score = 0;
        coinsCollected = 0;
        gameSpeed = 6;
        frameCount = 0;
        obstacles = [];
        coins = [];
        player.y = groundY - player.height;
        scoreText.innerText = 0;
        coinText.innerText = 0;
    }

    function handleJump(e) {
        if (gameState === 'PLAYING') {
            if (e.target && e.target.tagName === 'BUTTON') return;
            player.jump();
        }
    }

    window.addEventListener('keydown', (e) => {
        if (e.code === 'Space' || e.code === 'ArrowUp') handleJump(e);
    });

    gameWrapper.addEventListener('touchstart', (e) => {
        handleJump(e);
    }, { passive: false });

    gameWrapper.addEventListener('mousedown', (e) => {
        if (e.button === 0) handleJump(e);
    });

    startBtn.addEventListener('click', startGame);
    pauseBtn.addEventListener('click', pauseGame);
    resumeBtn.addEventListener('click', resumeGame);
    restartBtn.addEventListener('click', startGame);
    menuBtn1.addEventListener('click', showMenu);
    menuBtn2.addEventListener('click', showMenu);

    drawBackground();
</script>
</body>
</html>
