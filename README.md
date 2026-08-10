<html lang="bn">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Samba Runner BD - RRX STUDIOS</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; touch-action: manipulation; }
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: #0a0a0a;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            overflow: hidden;
            user-select: none;
        }

        #game-container {
            position: relative;
            width: 850px;
            height: 420px;
            box-shadow: 0 12px 40px rgba(0,0,0,0.9);
            border-radius: 16px;
            overflow: hidden;
            border: 4px solid #fed100;
        }

        canvas { display: block; width: 100%; height: 100%; }

        .overlay {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0, 0, 0, 0.82);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            color: #fff;
            z-index: 10;
            text-align: center;
            padding: 20px;
        }

        .studio-tag {
            font-size: 14px;
            letter-spacing: 3px;
            color: #00e676;
            text-transform: uppercase;
            font-weight: bold;
            margin-bottom: 5px;
        }

        h1 { font-size: 38px; color: #fed100; text-shadow: 3px 3px #009c3b; margin-bottom: 5px; }
        .edition-tag { font-size: 18px; color: #00b0ff; font-weight: bold; margin-bottom: 15px; }
        .sub-desc { font-size: 20px; margin-bottom: 25px; color: #ffeb3b; font-weight: 600; }

        .btn {
            padding: 12px 35px;
            font-size: 20px;
            font-weight: bold;
            color: #fff;
            background: linear-gradient(45deg, #009c3b, #002776);
            border: 2px solid #fed100;
            border-radius: 30px;
            cursor: pointer;
            box-shadow: 0 6px 20px rgba(0,0,0,0.5);
            transition: all 0.2s ease-in-out;
            outline: none;
            margin: 6px;
        }

        .btn:hover, .btn:active {
            transform: scale(1.06);
            box-shadow: 0 8px 25px rgba(254, 209, 0, 0.7);
            background: linear-gradient(45deg, #002776, #009c3b);
        }

        #hud {
            position: absolute;
            top: 15px; left: 20px; right: 20px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            z-index: 5;
            font-size: 18px;
            font-weight: bold;
            color: #fff;
            text-shadow: 2px 2px 4px #000;
        }

        .hud-btn { padding: 6px 16px; font-size: 14px; }
        .hidden { display: none !important; }
        
        /* Youtube Hidden Player */
        #audio-container { position: absolute; top: -9999px; left: -9999px; }
    </style>
</head>
<body>

<div id="game-container">
    <!-- Top HUD -->
    <div id="hud">
        <div>
            ⚽ Score: <span id="scoreText">0</span> | 
            🪙 Coins: <span id="coinText">0</span>
        </div>
        <div>
            <button id="pauseBtn" class="btn hud-btn">Pause</button>
        </div>
    </div>

    <canvas id="gameCanvas" width="850" height="420"></canvas>

    <!-- Start Menu Screen -->
    <div id="startScreen" class="overlay">
        <div class="studio-tag">RRX STUDIOS presents</div>
        <h1>Samba Runner BD ⚽</h1>
        <div class="edition-tag">🇧🇷 Brazil World Cup Edition 🇧🇷</div>
        <div class="sub-desc">7up খেও হেক্সা মিশন জিতো!</div>
        <button id="startBtn" class="btn">START GAME</button>
    </div>

    <!-- Pause Screen -->
    <div id="pauseScreen" class="overlay hidden">
        <h1>GAME PAUSED</h1>
        <button id="resumeBtn" class="btn">RESUME</button>
        <button id="menuBtn1" class="btn">MAIN MENU</button>
    </div>

    <!-- Game Over Screen -->
    <div id="gameOverScreen" class="overlay hidden">
        <h1 style="color: #ff4757;">ELIMINATED! 😭</h1>
        <p style="font-size: 20px; margin: 15px 0;">Final Score: <span id="finalScore">0</span> | Coins: <span id="finalCoins">0</span></p>
        <button id="restartBtn" class="btn">PLAY AGAIN</button>
        <button id="menuBtn2" class="btn">MAIN MENU</button>
    </div>
</div>

<!-- Background Youtube Funny Music Container -->
<div id="audio-container">
    <iframe id="yt-player" width="100" height="100" 
        src="https://www.youtube.com/embed/QtD4Wza458M?enablejsapi=1&loop=1&playlist=QtD4Wza458M" 
        allow="autoplay">
    </iframe>
</div>

<script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');

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
    const finalScore = document.getElementById('finalScore');
    const finalCoins = document.getElementById('finalCoins');

    // YouTube Audio API Controls
    let playerIframe = document.getElementById('yt-player');
    function playBgMusic() {
        playerIframe.contentWindow.postMessage('{"event":"command","func":"playVideo","args":""}', '*');
    }
    function pauseBgMusic() {
        playerIframe.contentWindow.postMessage('{"event":"command","func":"pauseVideo","args":""}', '*');
    }

    // Custom Web Audio Click
    const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    function playClickSound() {
        if(audioCtx.state === 'suspended') audioCtx.resume();
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        osc.type = 'triangle';
        osc.frequency.setValueAtTime(450, audioCtx.currentTime);
        osc.frequency.exponentialRampToValueAtTime(900, audioCtx.currentTime + 0.08);
        gain.gain.setValueAtTime(0.2, audioCtx.currentTime);
        gain.gain.linearRampToValueAtTime(0.01, audioCtx.currentTime + 0.08);
        osc.connect(gain);
        gain.connect(audioCtx.destination);
        osc.start();
        osc.stop(audioCtx.currentTime + 0.08);
    }

    // Game Variables
    let gameState = 'START'; 
    let score = 0;
    let coinsCollected = 0;
    let gameSpeed = 6;
    let frameCount = 0;

    const groundY = 340;
    const gravity = 0.65;

    // Player Object
    const player = {
        x: 80,
        y: groundY - 55,
        width: 38,
        height: 55,
        dy: 0,
        isJumping: false,
        draw() {
            // Legs running animation
            ctx.fillStyle = '#002776'; // Blue Shorts
            let legOffset = Math.sin(frameCount * 0.3) * 6;
            ctx.fillRect(this.x + 6 + (this.isJumping ? 0 : legOffset), this.y + 36, 10, 19);
            ctx.fillRect(this.x + 22 - (this.isJumping ? 0 : legOffset), this.y + 36, 10, 19);

            // Yellow Jersey
            ctx.fillStyle = '#fed100';
            ctx.fillRect(this.x, this.y + 14, this.width, 24);
            
            // Jersey Number 10
            ctx.fillStyle = '#002776';
            ctx.font = 'bold 12px Arial';
            ctx.fillText('10', this.x + 12, this.y + 31);

            // Green Collar
            ctx.fillStyle = '#009c3b';
            ctx.fillRect(this.x, this.y + 14, this.width, 4);

            // Head
            ctx.fillStyle = '#ffdbac';
            ctx.beginPath();
            ctx.arc(this.x + this.width / 2, this.y + 9, 10, 0, Math.PI * 2);
            ctx.fill();

            // Hair
            ctx.fillStyle = '#222';
            ctx.beginPath();
            ctx.arc(this.x + this.width / 2, this.y + 6, 10, Math.PI, Math.PI * 2);
            ctx.fill();
        },
        jump() {
            if (!this.isJumping) {
                this.dy = -13.5;
                this.isJumping = true;
                playClickSound();
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

    // Obstacles (7Up Cans & Spikes)
    let obstacles = [];
    class Obstacle {
        constructor() {
            this.x = canvas.width + 40;
            this.type = Math.random() > 0.4 ? '7UP' : 'SPIKE';
            this.width = this.type === '7UP' ? 32 : 36;
            this.height = this.type === '7UP' ? 52 : 32;
            this.y = groundY - this.height;
        }
        draw() {
            if (this.type === '7UP') {
                // 7Up Can
                ctx.fillStyle = '#00a651';
                ctx.fillRect(this.x, this.y, this.width, this.height);
                ctx.fillStyle = '#e0e0e0';
                ctx.fillRect(this.x, this.y, this.width, 5);
                ctx.fillStyle = '#fff';
                ctx.font = 'bold 13px Arial';
                ctx.fillText('7Up', this.x + 3, this.y + 26);
                ctx.fillStyle = '#ed1c24';
                ctx.beginPath();
                ctx.arc(this.x + 23, this.y + 36, 4, 0, Math.PI * 2);
                ctx.fill();
            } else {
                // Metal Spikes
                ctx.fillStyle = '#7f8c8d';
                ctx.beginPath();
                ctx.moveTo(this.x, groundY);
                ctx.lineTo(this.x + this.width/2, this.y);
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

    // Coins (Adjusted Height to easily catch!)
    let coins = [];
    class Coin {
        constructor() {
            this.x = canvas.width + 40;
            // Lowered height range for easier collection
            this.y = groundY - 60 - Math.random() * 45; 
            this.radius = 12;
        }
        draw() {
            ctx.fillStyle = '#ffd700';
            ctx.beginPath();
            ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
            ctx.fill();

            ctx.strokeStyle = '#ff8f00';
            ctx.lineWidth = 2;
            ctx.stroke();

            ctx.fillStyle = '#b8860b';
            ctx.font = 'bold 11px Arial';
            ctx.fillText('★', this.x - 4, this.y + 4);
        }
        update() {
            this.x -= gameSpeed;
            this.draw();
        }
    }

    // Background Rendering
    function drawBackground() {
        let skyGradient = ctx.createLinearGradient(0, 0, 0, 300);
        skyGradient.addColorStop(0, '#0a192f');
        skyGradient.addColorStop(1, '#1e3c72');
        ctx.fillStyle = skyGradient;
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        // Stadium Stand & Lights
        ctx.fillStyle = '#222';
        ctx.fillRect(0, 190, canvas.width, 100);
        
        ctx.fillStyle = 'rgba(255, 235, 59, 0.25)';
        for(let i = 20; i < canvas.width; i += 70) {
            ctx.fillRect(i, 200, 45, 90);
        }

        // Green Grass Field
        ctx.fillStyle = '#2e7d32';
        ctx.fillRect(0, groundY, canvas.width, canvas.height - groundY);

        // Moving Grass Striping
        ctx.strokeStyle = '#388e3c';
        ctx.lineWidth = 16;
        for (let i = (frameCount * -gameSpeed) % 60; i < canvas.width; i += 60) {
            ctx.beginPath();
            ctx.moveTo(i, groundY);
            ctx.lineTo(i, canvas.height);
            ctx.stroke();
        }

        // Top White Touchline
        ctx.strokeStyle = '#ffffff';
        ctx.lineWidth = 4;
        ctx.beginPath();
        ctx.moveTo(0, groundY);
        ctx.lineTo(canvas.width, groundY);
        ctx.stroke();
    }

    function handleSpawns() {
        if (frameCount % 95 === 0) obstacles.push(new Obstacle());
        if (frameCount % 130 === 0) coins.push(new Coin());
    }

    function checkCollisions() {
        for (let obs of obstacles) {
            if (
                player.x < obs.x + obs.width &&
                player.x + player.width > obs.x &&
                player.y < obs.y + obs.height &&
                player.y + player.height > obs.y
            ) {
                gameOver();
            }
        }

        coins.forEach((coin, index) => {
            let distX = (player.x + player.width/2) - coin.x;
            let distY = (player.y + player.height/2) - coin.y;
            let distance = Math.sqrt(distX * distX + distY * distY);

            if (distance < player.width/2 + coin.radius) {
                coinsCollected++;
                coinText.innerText = coinsCollected;
                playClickSound();
                coins.splice(index, 1);
            }
        });
    }

    function animate() {
        if (gameState !== 'PLAYING') return;

        ctx.clearRect(0, 0, canvas.width, canvas.height);
        frameCount++;

        drawBackground();
        handleSpawns();

        for (let i = obstacles.length - 1; i >= 0; i--) {
            obstacles[i].update();
            if (obstacles[i].x + obstacles[i].width < 0) obstacles.splice(i, 1);
        }

        for (let i = coins.length - 1; i >= 0; i--) {
            coins[i].update();
            if (coins[i].x + coins[i].radius < 0) coins.splice(i, 1);
        }

        player.update();
        checkCollisions();

        if (frameCount % 5 === 0) {
            score++;
            scoreText.innerText = score;
        }

        if (frameCount % 600 === 0) gameSpeed += 0.4;

        requestAnimationFrame(animate);
    }

    // Controls
    function handleJumpInput() {
        if (gameState === 'PLAYING') {
            player.jump();
        }
    }

    // Touch & Mouse Support
    canvas.addEventListener('touchstart', (e) => {
        e.preventDefault();
        handleJumpInput();
    });
    canvas.addEventListener('mousedown', (e) => {
        if (e.target === canvas) handleJumpInput();
    });
    window.addEventListener('keydown', (e) => {
        if (e.code === 'Space' || e.code === 'ArrowUp') handleJumpInput();
    });

    function startGame() {
        playClickSound();
        resetData();
        gameState = 'PLAYING';
        startScreen.classList.add('hidden');
        pauseScreen.classList.add('hidden');
        gameOverScreen.classList.add('hidden');
        playBgMusic();
        animate();
    }

    function pauseGame() {
        playClickSound();
        if (gameState === 'PLAYING') {
            gameState = 'PAUSED';
            pauseBgMusic();
            pauseScreen.classList.remove('hidden');
        }
    }

    function resumeGame() {
        playClickSound();
        gameState = 'PLAYING';
        pauseScreen.classList.add('hidden');
        playBgMusic();
        animate();
    }

    function gameOver() {
        gameState = 'GAMEOVER';
        pauseBgMusic();
        finalScore.innerText = score;
        finalCoins.innerText = coinsCollected;
        gameOverScreen.classList.remove('hidden');
    }

    function showMenu() {
        playClickSound();
        gameState = 'START';
        pauseBgMusic();
        resetData();
        startScreen.classList.remove('hidden');
        pauseScreen.classList.add('hidden');
        gameOverScreen.classList.add('hidden');
        ctx.clearRect(0, 0, canvas.width, canvas.height);
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
