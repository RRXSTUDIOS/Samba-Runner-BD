<html lang="bn">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Brazil Runner Game - Test</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: #111;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            overflow: hidden;
        }

        #game-container {
            position: relative;
            width: 800px;
            height: 400px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.8);
            border-radius: 12px;
            overflow: hidden;
            border: 4px solid #fed100;
        }

        canvas { display: block; }

        /* UI Overlays */
        .overlay {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0, 0, 0, 0.75);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            color: #fff;
            z-index: 10;
        }

        h1 { font-size: 42px; color: #fed100; text-shadow: 2px 2px #009c3b; margin-bottom: 10px; }
        p { font-size: 18px; margin-bottom: 20px; color: #ddd; }

        /* Fancy Animated Buttons */
        .btn {
            padding: 12px 30px;
            font-size: 20px;
            font-weight: bold;
            color: #fff;
            background: linear-gradient(45deg, #009c3b, #002776);
            border: 2px solid #fed100;
            border-radius: 25px;
            cursor: pointer;
            box-shadow: 0 5px 15px rgba(0,0,0,0.4);
            transition: all 0.2s ease-in-out;
            outline: none;
            margin: 5px;
        }

        .btn:hover {
            transform: translateY(-3px) scale(1.05);
            box-shadow: 0 8px 20px rgba(254, 209, 0, 0.6);
            background: linear-gradient(45deg, #002776, #009c3b);
        }

        .btn:active {
            transform: translateY(2px) scale(0.98);
        }

        #hud {
            position: absolute;
            top: 15px; left: 20px; right: 20px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            z-index: 5;
            font-size: 20px;
            font-weight: bold;
            color: #fff;
            text-shadow: 2px 2px 4px #000;
        }

        .hud-btn {
            padding: 6px 15px;
            font-size: 14px;
        }

        .hidden { display: none !important; }
    </style>
</head>
<body>

<div id="game-container">
    <!-- Top HUD (Score, Coins & Pause) -->
    <div id="hud">
        <div>
            ⚽ Score: <span id="scoreText">0</span> | 
            🪙 Coins: <span id="coinText">0</span>
        </div>
        <div>
            <button id="pauseBtn" class="btn hud-btn">Pause</button>
        </div>
    </div>

    <!-- Canvas Area -->
    <canvas id="gameCanvas" width="800" height="400"></canvas>

    <!-- Start Menu Screen -->
    <div id="startScreen" class="overlay">
        <h1>🇧🇷 BRAZIL DINO RUNNER ⚽</h1>
        <p>Space / Up Arrow চেপে লাফ দাও! 7Up এড়িয়ে কয়েন তোলো!</p>
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
        <p>Final Score: <span id="finalScore">0</span> | Coins: <span id="finalCoins">0</span></p>
        <button id="restartBtn" class="btn">PLAY AGAIN</button>
        <button id="menuBtn2" class="btn">MAIN MENU</button>
    </div>
</div>

<script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');

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
    const finalScore = document.getElementById('finalScore');
    const finalCoins = document.getElementById('finalCoins');

    // Web Audio API for Custom Generated Sounds (No External Audio Files Needed!)
    const audioCtx = new (window.AudioContext || window.webkitAudioContext)();

    function playClickSound() {
        if(audioCtx.state === 'suspended') audioCtx.resume();
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        osc.type = 'triangle';
        osc.frequency.setValueAtTime(400, audioCtx.currentTime);
        osc.frequency.exponentialRampToValueAtTime(800, audioCtx.currentTime + 0.1);
        gain.gain.setValueAtTime(0.3, audioCtx.currentTime);
        gain.gain.linearRampToValueAtTime(0.01, audioCtx.currentTime + 0.1);
        osc.connect(gain);
        gain.connect(audioCtx.destination);
        osc.start();
        osc.stop(audioCtx.currentTime + 0.1);
    }

    // Bangla Voice Generator Simulation for "Oi Dor Dor Chor" BG Audio Loop
    let bgSoundInterval;
    function startBgVoice() {
        if(bgSoundInterval) clearInterval(bgSoundInterval);
        bgSoundInterval = setInterval(() => {
            if (gameState === 'PLAYING') {
                const msg = new SpeechSynthesisUtterance("ঐ দৌড় দৌড় চোর");
                msg.lang = 'bn-BD';
                msg.pitch = 1.2;
                msg.rate = 1.3;
                window.speechSynthesis.speak(msg);
            }
        }, 2200);
    }

    function stopBgVoice() {
        if(bgSoundInterval) clearInterval(bgSoundInterval);
        window.speechSynthesis.cancel();
    }

    // Game Variables
    let gameState = 'START'; // START, PLAYING, PAUSED, GAMEOVER
    let score = 0;
    let coinsCollected = 0;
    let gameSpeed = 6;
    let frameCount = 0;

    // Ground & Gravity settings
    const groundY = 320;
    const gravity = 0.6;

    // Player Object (Brazil Jersey Theme)
    const player = {
        x: 80,
        y: groundY - 50,
        width: 35,
        height: 55,
        dy: 0,
        isJumping: false,
        draw() {
            // Legs (Running animation effect)
            ctx.fillStyle = '#002776'; // Blue Shorts
            ctx.fillRect(this.x + 5, this.y + 35, 10, 20);
            ctx.fillRect(this.x + 20, this.y + 35, 10, 20);

            // Brazil Jersey Body (Yellow)
            ctx.fillStyle = '#fed100';
            ctx.fillRect(this.x, this.y + 15, this.width, 25);
            
            // Green Collar & Details
            ctx.fillStyle = '#009c3b';
            ctx.fillRect(this.x, this.y + 15, this.width, 4);

            // Head & Face
            ctx.fillStyle = '#ffdbac';
            ctx.beginPath();
            ctx.arc(this.x + this.width / 2, this.y + 10, 10, 0, Math.PI * 2);
            ctx.fill();

            // Hair
            ctx.fillStyle = '#333';
            ctx.beginPath();
            ctx.arc(this.x + this.width / 2, this.y + 7, 10, Math.PI, Math.PI * 2);
            ctx.fill();
        },
        jump() {
            if (!this.isJumping) {
                this.dy = -13;
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

    // Obstacles (7Up Cans)
    let obstacles = [];
    class Obstacle {
        constructor() {
            this.x = canvas.width + 50;
            this.width = 30;
            this.height = 50;
            this.y = groundY - this.height;
        }
        draw() {
            // 7Up Can Green Body
            ctx.fillStyle = '#00a651';
            ctx.fillRect(this.x, this.y, this.width, this.height);

            // Can Top Silver Lid
            ctx.fillStyle = '#ccc';
            ctx.fillRect(this.x, this.y, this.width, 5);

            // 7Up Text Texture
            ctx.fillStyle = '#fff';
            ctx.font = 'bold 12px Arial';
            ctx.fillText('7Up', this.x + 3, this.y + 25);

            // Red Dot
            ctx.fillStyle = '#ed1c24';
            ctx.beginPath();
            ctx.arc(this.x + 22, this.y + 35, 4, 0, Math.PI * 2);
            ctx.fill();
        }
        update() {
            this.x -= gameSpeed;
            this.draw();
        }
    }

    // Coins (Golden)
    let coins = [];
    class Coin {
        constructor() {
            this.x = canvas.width + 50;
            this.y = groundY - 80 - Math.random() * 60;
            this.radius = 12;
        }
        draw() {
            ctx.fillStyle = '#ffd700';
            ctx.beginPath();
            ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
            ctx.fill();

            ctx.strokeStyle = '#daa520';
            ctx.lineWidth = 2;
            ctx.stroke();

            // Inner Star/Icon
            ctx.fillStyle = '#b8860b';
            ctx.font = 'bold 10px Arial';
            ctx.fillText('★', this.x - 4, this.y + 4);
        }
        update() {
            this.x -= gameSpeed;
            this.draw();
        }
    }

    // Background Rendering (World Cup Stadium & Grass Ground)
    function drawBackground() {
        // Stadium Sky Gradient
        let skyGradient = ctx.createLinearGradient(0, 0, 0, 300);
        skyGradient.addColorStop(0, '#0f2027');
        skyGradient.addColorStop(1, '#203a43');
        ctx.fillStyle = skyGradient;
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        // Stadium Stands & Lights
        ctx.fillStyle = '#333';
        ctx.fillRect(0, 180, canvas.width, 100);
        
        // Audience Lights effect
        ctx.fillStyle = 'rgba(255, 255, 255, 0.2)';
        for(let i = 20; i < canvas.width; i += 60) {
            ctx.fillRect(i, 190, 40, 80);
        }

        // Green Football Grass Ground
        ctx.fillStyle = '#2e7d32';
        ctx.fillRect(0, groundY, canvas.width, canvas.height - groundY);

        // Field Grass Lines (Football Pitch look)
        ctx.strokeStyle = '#4caf50';
        ctx.lineWidth = 15;
        for (let i = (frameCount * -gameSpeed) % 60; i < canvas.width; i += 60) {
            ctx.beginPath();
            ctx.moveTo(i, groundY);
            ctx.lineTo(i, canvas.height);
            ctx.stroke();
        }

        // White Ground Line
        ctx.strokeStyle = '#fff';
        ctx.lineWidth = 4;
        ctx.beginPath();
        ctx.moveTo(0, groundY);
        ctx.lineTo(canvas.width, groundY);
        ctx.stroke();
    }

    // Spawn Logic
    function handleSpawns() {
        if (frameCount % 100 === 0) {
            obstacles.push(new Obstacle());
        }
        if (frameCount % 150 === 0) {
            coins.push(new Coin());
        }
    }

    // Collision Detection
    function checkCollisions() {
        // Obstacle Collision
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

        // Coin Collection
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

    // Main Loop
    function animate() {
        if (gameState !== 'PLAYING') return;

        ctx.clearRect(0, 0, canvas.width, canvas.height);
        frameCount++;

        drawBackground();

        // Update & Draw Obstacles
        for (let i = obstacles.length - 1; i >= 0; i--) {
            obstacles[i].update();
            if (obstacles[i].x + obstacles[i].width < 0) obstacles.splice(i, 1);
        }

        // Update & Draw Coins
        for (let i = coins.length - 1; i >= 0; i--) {
            coins[i].update();
            if (coins[i].x + coins[i].radius < 0) coins.splice(i, 1);
        }

        player.update();
        checkCollisions();

        // Score Calculation
        if (frameCount % 5 === 0) {
            score++;
            scoreText.innerText = score;
        }

        // Speedup gradually
        if (frameCount % 500 === 0) gameSpeed += 0.5;

        requestAnimationFrame(animate);
    }

    // Game Control Functions
    function startGame() {
        playClickSound();
        resetData();
        gameState = 'PLAYING';
        startScreen.classList.add('hidden');
        pauseScreen.classList.add('hidden');
        gameOverScreen.classList.add('hidden');
        startBgVoice();
        animate();
    }

    function pauseGame() {
        playClickSound();
        if (gameState === 'PLAYING') {
            gameState = 'PAUSED';
            stopBgVoice();
            pauseScreen.classList.remove('hidden');
        }
    }

    function resumeGame() {
        playClickSound();
        gameState = 'PLAYING';
        pauseScreen.classList.add('hidden');
        startBgVoice();
        animate();
    }

    function gameOver() {
        gameState = 'GAMEOVER';
        stopBgVoice();
        finalScore.innerText = score;
        finalCoins.innerText = coinsCollected;
        gameOverScreen.classList.remove('hidden');
    }

    function showMenu() {
        playClickSound();
        gameState = 'START';
        stopBgVoice();
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

    // Event Listeners
    window.addEventListener('keydown', (e) => {
        if (e.code === 'Space' || e.code === 'ArrowUp') {
            if (gameState === 'PLAYING') player.jump();
        }
    });

    startBtn.addEventListener('click', startGame);
    pauseBtn.addEventListener('click', pauseGame);
    resumeBtn.addEventListener('click', resumeGame);
    restartBtn.addEventListener('click', startGame);
    menuBtn1.addEventListener('click', showMenu);
    menuBtn2.addEventListener('click', showMenu);

    // Initial Screen Draw
    drawBackground();
</script>
</body>
</html>
