<html lang="bn">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Samba Runner BD ⚽</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            font-family: 'Poppins', 'Segoe UI', sans-serif;
            background: #0a0f1d;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            overflow: hidden;
            color: #fff;
            user-select: none;
        }

        #game-wrapper {
            position: relative;
            width: 900px;
            height: 450px;
            box-shadow: 0 20px 50px rgba(0, 0, 0, 0.9), 0 0 30px rgba(254, 209, 0, 0.3);
            border-radius: 16px;
            overflow: hidden;
            border: 4px solid #fed100;
            background: #000;
            cursor: pointer;
        }

        canvas { display: block; width: 100%; height: 100%; }

        /* UI Screens */
        .overlay {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(10, 15, 29, 0.88);
            backdrop-filter: blur(8px);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 10;
            text-align: center;
            padding: 20px;
        }

        .title-container {
            margin-bottom: 20px;
            animation: bounce 2s infinite ease-in-out;
        }

        h1.game-title {
            font-size: 48px;
            font-weight: 900;
            background: linear-gradient(45deg, #fed100, #009c3b, #002776);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            text-shadow: 0 10px 20px rgba(0,0,0,0.5);
            letter-spacing: 2px;
        }

        /* Subtitle Style */
        .sub-title {
            font-size: 18px;
            color: #ffffff;
            margin-top: 6px;
            font-weight: 800;
            letter-spacing: 3px;
            text-transform: uppercase;
            text-shadow: 0 2px 10px rgba(255, 255, 255, 0.5);
        }

        /* Updated Instructions Style */
        p.instructions {
            font-size: 22px;
            font-weight: 800;
            margin-bottom: 25px;
            color: #ffffff;
            background: rgba(255, 255, 255, 0.12);
            padding: 12px 28px;
            border-radius: 30px;
            border: 2px solid rgba(254, 209, 0, 0.6);
            text-shadow: 0 2px 8px rgba(0, 0, 0, 0.8);
            letter-spacing: 1px;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.4);
        }

        /* Animated Buttons */
        .btn {
            padding: 14px 38px;
            font-size: 18px;
            font-weight: 800;
            color: #fff;
            background: linear-gradient(135deg, #009c3b 0%, #006837 100%);
            border: 2px solid #fed100;
            border-radius: 50px;
            cursor: pointer;
            box-shadow: 0 8px 25px rgba(0, 156, 59, 0.5);
            transition: all 0.25s cubic-bezier(0.175, 0.885, 0.32, 1.275);
            outline: none;
            margin: 6px;
            text-transform: uppercase;
            letter-spacing: 1px;
        }

        .btn:hover {
            transform: translateY(-4px) scale(1.05);
            box-shadow: 0 12px 30px rgba(254, 209, 0, 0.8);
            background: linear-gradient(135deg, #fed100 0%, #ff8c00 100%);
            color: #000;
            border-color: #009c3b;
        }

        .btn:active {
            transform: translateY(2px) scale(0.96);
        }

        /* Top HUD */
        #hud {
            position: absolute;
            top: 15px; left: 20px; right: 20px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            z-index: 5;
            font-size: 18px;
            font-weight: 700;
            color: #fff;
            pointer-events: none;
        }

        .hud-badge {
            background: rgba(0, 0, 0, 0.65);
            padding: 8px 16px;
            border-radius: 20px;
            border: 1px solid rgba(254, 209, 0, 0.4);
            display: inline-flex;
            align-items: center;
            gap: 8px;
            backdrop-filter: blur(4px);
        }

        .hud-btn {
            pointer-events: auto;
            padding: 8px 20px;
            font-size: 14px;
            border-radius: 20px;
        }

        .hidden { display: none !important; }

        @keyframes bounce {
            0%, 100% { transform: translateY(0); }
            50% { transform: translateY(-8px); }
        }
    </style>
</head>
<body>

<div id="game-wrapper">
    <!-- Hidden Container for YouTube Background Audio -->
    <div id="yt-player" style="position: absolute; width: 0; height: 0; opacity: 0; pointer-events: none;"></div>

    <!-- Heads Up Display (HUD) -->
    <div id="hud">
        <div style="display: flex; gap: 12px;">
            <div class="hud-badge">⚽ Score: <span id="scoreText" style="color: #fed100;">0</span></div>
            <div class="hud-badge">🪙 Coins: <span id="coinText" style="color: #ffd700;">0</span></div>
            <div class="hud-badge">🏆 High: <span id="highScoreText" style="color: #00ff87;">0</span></div>
        </div>
        <div>
            <button id="pauseBtn" class="btn hud-btn">Pause ⏸️</button>
        </div>
    </div>

    <!-- Canvas -->
    <canvas id="gameCanvas" width="900" height="450"></canvas>

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
        <h1 class="game-title" style="font-size: 36px;">GAME PAUSED ⏸️</h1>
        <br>
        <button id="resumeBtn" class="btn">RESUME ▶</button>
        <button id="menuBtn1" class="btn">MAIN MENU 🏠</button>
    </div>

    <!-- Game Over Screen -->
    <div id="gameOverScreen" class="overlay hidden">
        <h1 style="color: #ff4757; font-size: 42px; text-shadow: 0 5px 15px rgba(255,71,87,0.4);">ELIMINATED! 💥</h1>
        <p style="font-size: 20px; margin: 15px 0;">
            Final Score: <span id="finalScore" style="color: #fed100; font-weight: bold;">0</span> | 
            Coins: <span id="finalCoins" style="color: #ffd700; font-weight: bold;">0</span>
        </p>
        <div>
            <button id="restartBtn" class="btn">PLAY AGAIN 🔄</button>
            <button id="menuBtn2" class="btn">MAIN MENU 🏠</button>
        </div>
    </div>
</div>

<!-- YouTube IFrame API Script -->
<script src="https://www.youtube.com/iframe_api"></script>

<script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const gameWrapper = document.getElementById('game-wrapper');

    // UI Elements
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

    // High score from localStorage
    let highScore = localStorage.getItem('samba_runner_highscore') || 0;
    highScoreText.innerText = highScore;

    // Web Audio Synthesizer for Jump/Coin Sounds
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

    // YouTube Background Music Player (Looping)
    let playerYT;
    function onYouTubeIframeAPIReady() {
        playerYT = new YT.Player('yt-player', {
            height: '0',
            width: '0',
            videoId: 'QtD4Wza458M',
            playerVars: {
                'autoplay': 0,
                'controls': 0,
                'loop': 1,
                'playlist': 'QtD4Wza458M'
            }
        });
    }

    function playBGM() {
        if (playerYT && typeof playerYT.playVideo === 'function') {
            playerYT.playVideo();
        }
    }

    function pauseBGM() {
        if (playerYT && typeof playerYT.pauseVideo === 'function') {
            playerYT.pauseVideo();
        }
    }

    function stopBGM() {
        if (playerYT && typeof playerYT.stopVideo === 'function') {
            playerYT.stopVideo();
        }
    }

    // Game Variables
    let gameState = 'START';
    let score = 0;
    let coinsCollected = 0;
    let gameSpeed = 7;
    let frameCount = 0;

    const groundY = 360;
    const gravity = 0.65;

    // Player Object
    const player = {
        x: 90,
        y: groundY - 60,
        width: 38,
        height: 60,
        dy: 0,
        isJumping: false,
        legAngle: 0,
        
        draw() {
            ctx.save();
            ctx.translate(this.x + this.width / 2, this.y + this.height / 2);

            if (!this.isJumping) {
                this.legAngle = Math.sin(frameCount * 0.25) * 0.6;
            } else {
                this.legAngle = 0.4;
            }

            // Back Arm
            ctx.strokeStyle = '#e0ac69';
            ctx.lineWidth = 6;
            ctx.beginPath();
            ctx.moveTo(0, -10);
            ctx.lineTo(-Math.sin(this.legAngle) * 18, 5);
            ctx.stroke();

            // Back Leg
            ctx.strokeStyle = '#002776';
            ctx.lineWidth = 7;
            ctx.beginPath();
            ctx.moveTo(-4, 8);
            ctx.lineTo(-Math.sin(this.legAngle) * 20, 26);
            ctx.stroke();

            // Shoe Back
            ctx.fillStyle = '#fff';
            ctx.fillRect(-Math.sin(this.legAngle) * 20 - 3, 23, 10, 5);

            // Front Leg
            ctx.strokeStyle = '#002776';
            ctx.beginPath();
            ctx.moveTo(4, 8);
            ctx.lineTo(Math.sin(this.legAngle) * 20, 26);
            ctx.stroke();

            // Shoe Front
            ctx.fillStyle = '#fff';
            ctx.fillRect(Math.sin(this.legAngle) * 20 - 3, 23, 10, 5);

            // Body Jersey (Brazil Yellow)
            ctx.fillStyle = '#fed100';
            ctx.beginPath();
            ctx.roundRect(-14, -18, 28, 28, 4);
            ctx.fill();

            // Jersey Green Collar & Details
            ctx.fillStyle = '#009c3b';
            ctx.fillRect(-14, -18, 28, 5);
            ctx.fillRect(-2, -13, 4, 18);

            // Jersey Number 10
            ctx.fillStyle = '#002776';
            ctx.font = 'bold 11px sans-serif';
            ctx.textAlign = 'center';
            ctx.fillText('10', 0, -2);

            // Front Arm
            ctx.strokeStyle = '#e0ac69';
            ctx.lineWidth = 6;
            ctx.beginPath();
            ctx.moveTo(0, -10);
            ctx.lineTo(Math.sin(this.legAngle) * 18, 5);
            ctx.stroke();

            // Head
            ctx.fillStyle = '#e0ac69';
            ctx.beginPath();
            ctx.arc(0, -28, 11, 0, Math.PI * 2);
            ctx.fill();

            // Hair
            ctx.fillStyle = '#222';
            ctx.beginPath();
            ctx.arc(0, -31, 11, Math.PI * 0.8, Math.PI * 2.2);
            ctx.fill();

            // Headband (Green)
            ctx.fillStyle = '#009c3b';
            ctx.fillRect(-11, -33, 22, 4);

            ctx.restore();
        },
        
        jump() {
            if (!this.isJumping) {
                this.dy = -14.5;
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

    // Obstacles
    let obstacles = [];
    class Obstacle {
        constructor() {
            this.type = Math.random() > 0.4 ? '7UP' : 'SPIKE';
            this.x = canvas.width + 40;
            
            if (this.type === '7UP') {
                this.width = 32;
                this.height = 54;
            } else {
                this.width = 36;
                this.height = 38;
            }
            this.y = groundY - this.height;
        }

        draw() {
            if (this.type === '7UP') {
                let gradient = ctx.createLinearGradient(this.x, 0, this.x + this.width, 0);
                gradient.addColorStop(0, '#00833e');
                gradient.addColorStop(0.5, '#00a651');
                gradient.addColorStop(1, '#005826');
                ctx.fillStyle = gradient;
                ctx.beginPath();
                ctx.roundRect(this.x, this.y, this.width, this.height, 4);
                ctx.fill();

                ctx.fillStyle = '#e6e6e6';
                ctx.fillRect(this.x + 2, this.y, this.width - 4, 4);
                ctx.fillRect(this.x + 2, this.y + this.height - 4, this.width - 4, 4);

                ctx.fillStyle = '#ffffff';
                ctx.font = 'bold 13px sans-serif';
                ctx.fillText('7 Up', this.x + 2, this.y + 28);

                ctx.fillStyle = '#ed1c24';
                ctx.beginPath();
                ctx.arc(this.x + 25, this.y + 36, 4, 0, Math.PI * 2);
                ctx.fill();

            } else {
                ctx.fillStyle = '#d63031';
                ctx.strokeStyle = '#2d3436';
                ctx.lineWidth = 2;

                ctx.beginPath();
                ctx.moveTo(this.x, groundY);
                ctx.lineTo(this.x + 6, groundY - this.height);
                ctx.lineTo(this.x + 12, groundY);

                ctx.lineTo(this.x + 18, groundY - this.height - 6);
                ctx.lineTo(this.x + 24, groundY);

                ctx.lineTo(this.x + 30, groundY - this.height + 4);
                ctx.lineTo(this.x + this.width, groundY);
                ctx.closePath();
                ctx.fill();
                ctx.stroke();
            }
        }

        update() {
            this.x -= gameSpeed;
            this.draw();
        }
    }

    // 3D Animated Golden Coins
    let coins = [];
    class Coin {
        constructor() {
            this.x = canvas.width + 30;
            this.y = groundY - 45 - Math.random() * 55;
            this.radius = 13;
            this.spinVal = Math.random() * Math.PI;
        }

        draw() {
            this.spinVal += 0.08;
            let widthFactor = Math.abs(Math.sin(this.spinVal));

            ctx.save();
            ctx.translate(this.x, this.y);
            ctx.scale(widthFactor, 1);

            ctx.shadowColor = '#ffd700';
            ctx.shadowBlur = 10;

            ctx.fillStyle = '#ffd700';
            ctx.beginPath();
            ctx.arc(0, 0, this.radius, 0, Math.PI * 2);
            ctx.fill();

            ctx.strokeStyle = '#b8860b';
            ctx.lineWidth = 2;
            ctx.stroke();

            ctx.fillStyle = '#d4af37';
            ctx.font = 'bold 11px sans-serif';
            ctx.textAlign = 'center';
            ctx.textBaseline = 'middle';
            ctx.fillText('⚽', 0, 0);

            ctx.restore();
        }

        update() {
            this.x -= gameSpeed;
            this.draw();
        }
    }

    // Particle Burst Effects
    let particles = [];
    function createParticles(x, y, color) {
        for (let i = 0; i < 12; i++) {
            particles.push({
                x: x, y: y,
                dx: (Math.random() - 0.5) * 6,
                dy: (Math.random() - 0.5) * 6,
                size: Math.random() * 4 + 2,
                color: color,
                life: 20
            });
        }
    }

    function updateParticles() {
        for (let i = particles.length - 1; i >= 0; i--) {
            let p = particles[i];
            p.x += p.dx;
            p.y += p.dy;
            p.life--;

            ctx.fillStyle = p.color;
            ctx.beginPath();
            ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
            ctx.fill();

            if (p.life <= 0) particles.splice(i, 1);
        }
    }

    // Background Rendering
    let adOffset = 0;
    function drawStadiumBackground() {
        let skyGrad = ctx.createLinearGradient(0, 0, 0, 280);
        skyGrad.addColorStop(0, '#050b14');
        skyGrad.addColorStop(0.6, '#0f1f38');
        skyGrad.addColorStop(1, '#1b3459');
        ctx.fillStyle = skyGrad;
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        // Floodlights
        ctx.fillStyle = 'rgba(255, 255, 255, 0.08)';
        ctx.beginPath();
        ctx.moveTo(100, 0); ctx.lineTo(0, 280); ctx.lineTo(300, 280); ctx.closePath();
        ctx.fill();

        ctx.beginPath();
        ctx.moveTo(800, 0); ctx.lineTo(600, 280); ctx.lineTo(900, 280); ctx.closePath();
        ctx.fill();

        ctx.fillStyle = '#fff';
        ctx.shadowColor = '#fff';
        ctx.shadowBlur = 15;
        for (let x = 80; x <= 140; x += 20) ctx.fillRect(x, 10, 12, 12);
        for (let x = 760; x <= 820; x += 20) ctx.fillRect(x, 10, 12, 12);
        ctx.shadowBlur = 0;

        // Audience Stands
        ctx.fillStyle = '#111827';
        ctx.fillRect(0, 160, canvas.width, 110);

        for (let i = 0; i < 40; i++) {
            if (Math.random() > 0.85) {
                let cx = (i * 23 + frameCount * 2) % canvas.width;
                let cy = 170 + (i % 4) * 20;
                ctx.fillStyle = Math.random() > 0.5 ? '#fed100' : '#ffffff';
                ctx.fillRect(cx, cy, 3, 3);
            }
        }

        // LED Ad Boards
        adOffset += gameSpeed * 0.5;
        ctx.fillStyle = '#000';
        ctx.fillRect(0, 270, canvas.width, 30);
        ctx.strokeStyle = '#fed100';
        ctx.lineWidth = 2;
        ctx.strokeRect(0, 270, canvas.width, 30);

        ctx.fillStyle = '#00ff87';
        ctx.font = 'bold 13px sans-serif';
        let adText = "  ⚽ SAMBA RUNNER BD  |  RRX STUDIOS PRESENTS  |  7 আপ খাও হেক্সা মিশন জিতো  |  GOLDEN BOOT RUN  ";
        let textWidth = ctx.measureText(adText).width;
        let xPos = -(adOffset % textWidth);
        ctx.fillText(adText + adText, xPos, 290);

        // Turf Ground
        ctx.fillStyle = '#1e7e34';
        ctx.fillRect(0, groundY, canvas.width, canvas.height - groundY);

        let stripeWidth = 50;
        let offset = (frameCount * gameSpeed) % (stripeWidth * 2);
        ctx.fillStyle = '#28a745';
        for (let x = -offset; x < canvas.width + stripeWidth; x += stripeWidth * 2) {
            ctx.fillRect(x, groundY, stripeWidth, canvas.height - groundY);
        }

        ctx.fillStyle = '#ffffff';
        ctx.fillRect(0, groundY, canvas.width, 4);
    }

    // Spawn Logic
    function handleSpawns() {
        if (frameCount % 90 === 0) {
            obstacles.push(new Obstacle());
        }
        if (frameCount % 120 === 0) {
            coins.push(new Coin());
        }
    }

    // Collision Detection
    function checkCollisions() {
        for (let obs of obstacles) {
            let hitBoxPadding = 6;
            if (
                player.x + hitBoxPadding < obs.x + obs.width &&
                player.x + player.width - hitBoxPadding > obs.x &&
                player.y + hitBoxPadding < obs.y + obs.height &&
                player.y + player.height > obs.y
            ) {
                createParticles(player.x + player.width/2, player.y + player.height/2, '#d63031');
                playSound('hit');
                gameOver();
            }
        }

        for (let i = coins.length - 1; i >= 0; i--) {
            let coin = coins[i];
            let dx = (player.x + player.width/2) - coin.x;
            let dy = (player.y + player.height/2) - coin.y;
            let dist = Math.sqrt(dx * dx + dy * dy);

            if (dist < player.width/2 + coin.radius) {
                coinsCollected++;
                coinText.innerText = coinsCollected;
                createParticles(coin.x, coin.y, '#ffd700');
                playSound('coin');
                coins.splice(i, 1);
            }
        }
    }

    // Main Game Loop
    function animate() {
        if (gameState !== 'PLAYING') return;

        ctx.clearRect(0, 0, canvas.width, canvas.height);
        frameCount++;

        drawStadiumBackground();
        handleSpawns();

        for (let i = obstacles.length - 1; i >= 0; i--) {
            obstacles[i].update();
            if (obstacles[i].x + obstacles[i].width < -20) obstacles.splice(i, 1);
        }

        for (let i = coins.length - 1; i >= 0; i--) {
            coins[i].update();
            if (coins[i].x + coins[i].radius < -20) coins.splice(i, 1);
        }

        player.update();
        updateParticles();
        checkCollisions();

        if (frameCount % 6 === 0) {
            score++;
            scoreText.innerText = score;
        }

        if (frameCount % 400 === 0) gameSpeed += 0.4;

        requestAnimationFrame(animate);
    }

    // Game States & Music Handlers
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
        drawStadiumBackground();
    }

    function resetData() {
        score = 0;
        coinsCollected = 0;
        gameSpeed = 7;
        frameCount = 0;
        obstacles = [];
        coins = [];
        particles = [];
        player.y = groundY - player.height;
        scoreText.innerText = 0;
        coinText.innerText = 0;
    }

    // Input Control Handlers
    function handleJump(e) {
        if (gameState === 'PLAYING') {
            if (e.target && e.target.id === 'pauseBtn') return;
            player.jump();
        }
    }

    window.addEventListener('keydown', (e) => {
        if (e.code === 'Space' || e.code === 'ArrowUp') {
            e.preventDefault();
            handleJump(e);
        }
    });

    gameWrapper.addEventListener('touchstart', (e) => {
        if (gameState === 'PLAYING' && e.target.tagName !== 'BUTTON') {
            e.preventDefault();
            handleJump(e);
        }
    });

    gameWrapper.addEventListener('mousedown', (e) => {
        if (e.button === 0 && gameState === 'PLAYING' && e.target.tagName !== 'BUTTON') {
            handleJump(e);
        }
    });

    startBtn.addEventListener('click', startGame);
    pauseBtn.addEventListener('click', pauseGame);
    resumeBtn.addEventListener('click', resumeGame);
    restartBtn.addEventListener('click', startGame);
    menuBtn1.addEventListener('click', showMenu);
    menuBtn2.addEventListener('click', showMenu);

    drawStadiumBackground();
</script>
</body>
</html>
