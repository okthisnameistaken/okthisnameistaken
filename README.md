<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Galactic RPG Invaders</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700&display=swap');
        body {
            background-color: #0d1117;
            color: #ffffff;
            font-family: 'Orbitron', sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            padding: 10px;
        }
        #gameContainer {
            background: #161b22;
            border: 4px solid #00ff99;
            border-radius: 8px;
            box-shadow: 0 0 20px rgba(0, 255, 153, 0.5);
            padding: 10px;
            display: flex;
            flex-direction: column;
            gap: 10px;
            max-width: 900px;
            width: 100%;
        }
        canvas {
            display: block;
            background: #010409;
            border: 2px solid #00774c;
            /* Ensures canvas is responsive within its container */
            width: 100%;
            max-width: 800px;
            margin: 0 auto;
        }
        .hud-item, .stat-value {
            font-size: 1.1rem;
            color: #00ff99;
        }
        .btn-game {
            background-color: #00ff99;
            color: #0d1117;
            font-weight: bold;
            padding: 8px 16px;
            border-radius: 6px;
            cursor: pointer;
            transition: background-color 0.2s, transform 0.1s;
        }
        .btn-game:hover {
            background-color: #00e68a;
            transform: translateY(-1px);
        }
        .btn-game:active {
            transform: translateY(1px);
        }
        .progress-bar-container {
            width: 100%;
            height: 12px;
            background-color: #333;
            border-radius: 6px;
            overflow: hidden;
            border: 1px solid #00ff99;
        }
        .progress-bar {
            height: 100%;
            background-color: #00ff99;
            transition: width 0.3s;
        }
        .modal {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.9);
            display: flex;
            justify-content: center;
            align-items: center;
            z-index: 100;
        }
        .modal-content {
            background: #161b22;
            border: 3px solid #00ff99;
            border-radius: 10px;
            padding: 20px;
            width: 90%;
            max-width: 500px;
            box-shadow: 0 0 30px rgba(0, 255, 153, 0.7);
        }
        .upgrade-item {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 10px;
            margin-bottom: 8px;
            background-color: #0d1117;
            border-radius: 4px;
        }
    </style>
</head>
<body>

<div id="gameContainer">
    <!-- HUD: Heads-Up Display for RPG Stats -->
    <div class="p-2 border border-gray-700 rounded-md flex justify-between flex-wrap text-sm" id="hud">
        <div class="flex items-center gap-2">
            <span class="hud-item text-red-500">HP:</span>
            <div class="w-24 progress-bar-container">
                <div class="progress-bar bg-red-500" id="hpBar" style="width: 100%;"></div>
            </div>
        </div>
        <div class="flex items-center gap-2">
            <span class="hud-item text-yellow-400">XP:</span>
            <div class="w-24 progress-bar-container">
                <div class="progress-bar bg-yellow-400" id="xpBar" style="width: 0%;"></div>
            </div>
        </div>
        <span class="hud-item">LVL: <span class="stat-value" id="levelValue">1</span></span>
        <span class="hud-item">GOLD: <span class="stat-value text-yellow-500" id="goldValue">0</span></span>
        <span class="hud-item">SP: <span class="stat-value text-blue-400" id="spValue">0</span></span>
        <span class="hud-item">WAVE: <span class="stat-value" id="waveValue">1</span></span>
    </div>

    <!-- Game Canvas -->
    <canvas id="gameCanvas" width="800" height="600"></canvas>

    <!-- Controls and Messages -->
    <div class="p-2 flex justify-between items-center">
        <!-- UPDATED: Displaying keyboard controls only -->
        <span class="text-sm text-gray-400">Controls: ←, → (or A/D) to move, ↑ to shoot. 'U' to Upgrade.</span>
        <button id="upgradeButton" class="btn-game">
            <span id="upgradeText">UPGRADE (U)</span>
        </button>
    </div>
</div>

<!-- Upgrade Shop Modal (hidden by default) -->
<div id="upgradeModal" class="modal hidden">
    <div class="modal-content">
        <h2 class="text-2xl text-center mb-6 text-[#00ff99] border-b pb-2 border-gray-700">Upgrade Terminal</h2>

        <div class="mb-4 text-center">
            <p class="text-lg">Skill Points (SP): <span class="stat-value text-blue-400" id="modalSpValue">0</span></p>
        </div>

        <div id="upgradesList">
            <!-- Damage Upgrade -->
            <div class="upgrade-item">
                <div>
                    <span class="text-lg text-red-400">Damage +1</span> (Current: <span id="currentDamage">1</span>)
                    <p class="text-xs text-gray-500">Increase weapon power.</p>
                </div>
                <button class="btn-game text-sm" data-stat="damage" data-cost="1">Cost: 1 SP</button>
            </div>
            <!-- Fire Rate Upgrade -->
            <div class="upgrade-item">
                <div>
                    <span class="text-lg text-cyan-400">Fire Rate +5%</span> (Current: <span id="currentFireRate">100ms</span>)
                    <p class="text-xs text-gray-500">Reduce shot cooldown.</p>
                </div>
                <button class="btn-game text-sm" data-stat="fireRate" data-cost="2">Cost: 2 SP</button>
            </div>
            <!-- Max Health Upgrade -->
            <div class="upgrade-item">
                <div>
                    <span class="text-lg text-lime-400">Max Health +10</span> (Current: <span id="currentMaxHealth">100</span>)
                    <p class="text-xs text-gray-500">Improve ship durability.</p>
                </div>
                <button class="btn-game text-sm" data-stat="maxHealth" data-cost="1">Cost: 1 SP</button>
            </div>
            <!-- Health Pack Purchase -->
            <div class="upgrade-item">
                <div>
                    <span class="text-lg text-yellow-500">Health Pack</span> (Heals 50 HP)
                    <p class="text-xs text-gray-500">Restore vital hull integrity.</p>
                </div>
                <button class="btn-game text-sm" data-stat="heal" data-cost="50">Cost: 50 GOLD</button>
            </div>
        </div>

        <button id="closeModalButton" class="btn-game w-full mt-6">Close Terminal (U)</button>
    </div>
</div>

<script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    
    // Set fixed canvas size for game logic consistency
    const GAME_WIDTH = 800;
    const GAME_HEIGHT = 600;
    canvas.width = GAME_WIDTH;
    canvas.height = GAME_HEIGHT;

    // --- GAME STATE ---
    let game = {
        isRunning: true,
        isPaused: false,
        wave: 1,
        waveProgress: 0, // Number of enemies spawned in current wave
        waveSize: 10,
        spawnCooldown: 120, // Frames between enemy spawns
        spawnTimer: 0,
        gameOver: false,
        lastSpawnY: 0,
    };

    let player = {
        x: GAME_WIDTH / 2,
        y: GAME_HEIGHT - 60,
        size: 30,
        speed: 5,
        health: 100,
        maxHealth: 100,
        damage: 1,
        fireRate: 200, // Cooldown in milliseconds
        fireCooldown: 0,
        level: 1,
        xp: 0,
        xpToNextLevel: 10,
        gold: 0,
        skillPoints: 0,
    };

    let entities = {
        bullets: [],
        enemies: [],
        enemyBullets: [],
        explosions: [],
    };

    let input = {
        left: false,
        right: false,
        shoot: false,
    };
    
    // --- UTILITIES ---

    const calculateXpNeeded = (level) => {
        return Math.floor(10 * Math.pow(1.5, level - 1));
    };

    const clamp = (num, min, max) => Math.min(Math.max(num, min), max);

    // --- ENTITY CLASSES ---

    class Bullet {
        constructor(x, y, color, speedY, damage, owner) {
            this.x = x;
            this.y = y;
            this.width = 3;
            this.height = 10;
            this.color = color;
            this.speedY = speedY;
            this.damage = damage;
            this.owner = owner; // 'player' or 'enemy'
        }

        update() {
            this.y += this.speedY;
        }

        draw() {
            ctx.fillStyle = this.color;
            ctx.fillRect(this.x - this.width / 2, this.y, this.width, this.height);
        }

        isOffScreen() {
            return this.y < -this.height || this.y > GAME_HEIGHT;
        }
    }

    class Enemy {
        constructor(x, y, type) {
            this.x = x;
            this.y = y;
            this.type = type; // 'Basic', 'Tank', 'Fast'
            this.width = 40;
            this.height = 30;
            this.speedX = 1;
            this.speedY = 0.05;
            this.direction = 1; // 1 = right, -1 = left
            this.shotChance = 0.001 * game.wave; // Increased chance per wave

            // RPG Stats for Enemies based on type and wave
            this.stats = this.getEnemyStats(type);
            this.health = this.stats.health * game.wave;
            this.maxHealth = this.health;
            this.xpValue = this.stats.xpValue * game.wave;
            this.goldValue = this.stats.goldValue * game.wave;
            this.color = this.stats.color;
        }

        getEnemyStats(type) {
            switch (type) {
                case 'Tank':
                    return { health: 15, damage: 2, xpValue: 3, goldValue: 5, color: '#ff6347' }; // Tomato Red
                case 'Fast':
                    return { health: 5, damage: 1, xpValue: 1, goldValue: 3, color: '#ffd700' }; // Gold
                case 'Basic':
                default:
                    return { health: 10, damage: 1, xpValue: 2, goldValue: 4, color: '#1e90ff' }; // Dodger Blue
            }
        }

        update() {
            this.x += this.speedX * this.direction * (this.type === 'Fast' ? 1.5 : 1);
            this.y += this.speedY;

            // Boundary check and change direction
            if (this.x + this.width > GAME_WIDTH || this.x < 0) {
                this.direction *= -1;
                this.x = clamp(this.x, 0, GAME_WIDTH - this.width);
                this.y += 5; // Drop down slightly when hitting the wall
            }

            // Shooting logic
            if (Math.random() < this.shotChance) {
                entities.enemyBullets.push(new Bullet(
                    this.x + this.width / 2, 
                    this.y + this.height, 
                    '#ff0000', 
                    4, 
                    this.stats.damage, 
                    'enemy'
                ));
            }
        }

        draw() {
            // Draw Enemy Ship
            ctx.fillStyle = this.color;
            ctx.fillRect(this.x, this.y, this.width, this.height);

            // Draw HP Bar
            const hpWidth = this.width * (this.health / this.maxHealth);
            ctx.fillStyle = '#ff4444';
            ctx.fillRect(this.x, this.y - 8, this.width, 4);
            ctx.fillStyle = '#44ff44';
            ctx.fillRect(this.x, this.y - 8, hpWidth, 4);
        }
    }

    // --- GAME LOGIC ---

    function initGame() {
        game.isRunning = true;
        game.isPaused = false;
        game.gameOver = false;
        game.wave = 1;
        game.waveProgress = 0;
        game.waveSize = 10;
        game.spawnTimer = 0;
        
        // Reset player to initial state
        player.health = player.maxHealth;
        player.x = GAME_WIDTH / 2;
        player.y = GAME_HEIGHT - 60;

        entities.bullets = [];
        entities.enemies = [];
        entities.enemyBullets = [];
        entities.explosions = [];

        hideModal();
    }

    function checkLevelUp() {
        while (player.xp >= player.xpToNextLevel) {
            player.level++;
            player.xp -= player.xpToNextLevel;
            player.xpToNextLevel = calculateXpNeeded(player.level);
            player.skillPoints++;
            
            // Notification or visual effect for level up
            createFloatingText(`LEVEL UP! LVL ${player.level}`, GAME_WIDTH / 2, GAME_HEIGHT / 2, 60, '#00ff99');
        }
    }

    function spawnEnemy() {
        // Randomize enemy type based on wave (more variety/toughness in higher waves)
        let enemyTypes = ['Basic'];
        if (game.wave >= 2) enemyTypes.push('Fast');
        if (game.wave >= 3) enemyTypes.push('Tank');
        
        const typeIndex = Math.floor(Math.random() * enemyTypes.length);
        const type = enemyTypes[typeIndex];

        // Ensure enemies don't overlap too much
        const minSpacing = 50;
        let x = Math.random() * (GAME_WIDTH - 40);
        x = clamp(x, 40, GAME_WIDTH - 80);
        
        // Ensure enemies spawn at the top
        const y = Math.random() * 50;
        
        entities.enemies.push(new Enemy(x, y, type));
        game.waveProgress++;
        game.spawnTimer = game.spawnCooldown - Math.min(game.wave * 10, game.spawnCooldown - 30); // Speed up spawning per wave
        game.lastSpawnY = y;
    }

    function startNextWave() {
        game.wave++;
        game.waveProgress = 0;
        game.waveSize = 10 + game.wave * 5; // Increase wave size
        
        // Add a temporary text notification
        createFloatingText(`WAVE ${game.wave} INCOMING!`, GAME_WIDTH / 2, GAME_HEIGHT / 2 - 50, 80, '#ffff00');
    }

    function playerShoot() {
        if (player.fireCooldown <= 0) {
            entities.bullets.push(new Bullet(
                player.x, 
                player.y - player.size / 2, 
                '#00ff99', 
                -8, 
                player.damage, 
                'player'
            ));
            player.fireCooldown = player.fireRate;
        }
    }

    // --- COLLISION DETECTION ---

    function isColliding(a, b) {
        return a.x < b.x + b.width &&
               a.x + a.width > b.x &&
               a.y < b.y + b.height &&
               a.y + a.height > b.y;
    }
    
    // --- DRAWING AND HUD ---

    function drawPlayer() {
        // Draw Ship (simple triangle)
        ctx.fillStyle = '#00ffff'; // Cyan
        ctx.beginPath();
        ctx.moveTo(player.x, player.y - player.size);
        ctx.lineTo(player.x + player.size / 2, player.y);
        ctx.lineTo(player.x - player.size / 2, player.y);
        ctx.closePath();
        ctx.fill();
    }

    function updateHUD() {
        document.getElementById('levelValue').textContent = player.level;
        document.getElementById('goldValue').textContent = player.gold;
        document.getElementById('spValue').textContent = player.skillPoints;
        document.getElementById('waveValue').textContent = game.wave;

        // HP Bar
        const hpPercent = clamp((player.health / player.maxHealth) * 100, 0, 100);
        document.getElementById('hpBar').style.width = `${hpPercent}%`;
        
        // XP Bar
        const xpPercent = clamp((player.xp / player.xpToNextLevel) * 100, 0, 100);
        document.getElementById('xpBar').style.width = `${xpPercent}%`;

        // Update Upgrade Button text
        document.getElementById('upgradeText').textContent = player.skillPoints > 0 ? `UPGRADE (${player.skillPoints} SP READY)` : 'UPGRADE (U)';
    }

    function createFloatingText(text, x, y, duration, color) {
        entities.explosions.push({
            text: text,
            x: x,
            y: y,
            duration: duration,
            color: color,
            life: 0,
            speedY: -0.5,
        });
    }

    function drawFloatingText() {
        for (let i = 0; i < entities.explosions.length; i++) {
            const exp = entities.explosions[i];
            
            exp.life++;
            exp.y += exp.speedY;
            
            const alpha = 1 - (exp.life / exp.duration);

            ctx.font = `bold ${20 + (1 - alpha) * 10}px 'Orbitron'`;
            ctx.fillStyle = `${exp.color}`;
            ctx.textAlign = 'center';
            ctx.fillText(exp.text, exp.x, exp.y);
            
            if (exp.life >= exp.duration) {
                entities.explosions.splice(i, 1);
                i--;
            }
        }
    }


    // --- MAIN GAME LOOP ---

    function update() {
        if (!game.isRunning || game.isPaused || game.gameOver) return;

        // 1. Player Movement and Shooting
        if (input.left) player.x -= player.speed;
        if (input.right) player.x += player.speed;
        player.x = clamp(player.x, player.size / 2, GAME_WIDTH - player.size / 2);
        
        if (input.shoot) playerShoot();

        // Update Cooldowns
        player.fireCooldown -= 1000 / 60; // Assuming 60 FPS

        // 2. Wave and Spawn Logic
        if (entities.enemies.length === 0 && game.waveProgress >= game.waveSize) {
            startNextWave();
        }

        if (game.waveProgress < game.waveSize) {
            game.spawnTimer--;
            if (game.spawnTimer <= 0) {
                spawnEnemy();
            }
        }

        // 3. Update Entities
        entities.bullets.forEach(b => b.update());
        entities.enemyBullets.forEach(b => b.update());
        entities.enemies.forEach(e => e.update());

        // 4. Collision Detection
        
        // Player Bullets vs. Enemies
        for (let i = 0; i < entities.bullets.length; i++) {
            const bullet = entities.bullets[i];
            
            for (let j = 0; j < entities.enemies.length; j++) {
                const enemy = entities.enemies[j];
                
                if (isColliding(bullet, enemy)) {
                    enemy.health -= bullet.damage;
                    
                    if (enemy.health <= 0) {
                        // RPG Reward
                        player.xp += enemy.xpValue;
                        player.gold += enemy.goldValue;
                        createFloatingText(`+${enemy.xpValue} XP, +${enemy.goldValue} Gold`, enemy.x + enemy.width / 2, enemy.y, 40, '#ffbb00');

                        // Remove enemy and check for level up
                        entities.enemies.splice(j, 1);
                        j--;
                        checkLevelUp();
                    } else {
                        // Damage visual text
                        createFloatingText(`-${bullet.damage}`, bullet.x, bullet.y, 20, '#ffffff');
                    }
                    
                    // Remove bullet
                    entities.bullets.splice(i, 1);
                    i--;
                    break;
                }
            }
        }
        
        // Enemy Bullets vs. Player
        for (let i = 0; i < entities.enemyBullets.length; i++) {
            const bullet = entities.enemyBullets[i];
            
            // Create a temporary rectangle for player collision check
            const playerCollision = {
                x: player.x - player.size / 2, 
                y: player.y - player.size, 
                width: player.size, 
                height: player.size
            };
            
            if (isColliding(bullet, playerCollision)) {
                player.health -= bullet.damage;
                player.health = clamp(player.health, 0, player.maxHealth);

                // Damage visual text
                createFloatingText(`-${bullet.damage} HP`, player.x, player.y - player.size - 10, 40, '#ff4444');
                
                entities.enemyBullets.splice(i, 1);
                i--;

                if (player.health <= 0) {
                    game.gameOver = true;
                    // Game Over logic will run in the draw function
                    return;
                }
            }
        }
        
        // Remove off-screen bullets
        entities.bullets = entities.bullets.filter(b => !b.isOffScreen());
        entities.enemyBullets = entities.enemyBullets.filter(b => !b.isOffScreen());
        
        // 5. Check if enemies reached the player's level
        if (entities.enemies.some(e => e.y + e.height > GAME_HEIGHT - 60)) {
            game.gameOver = true;
            return;
        }
    }

    function draw() {
        // Clear canvas
        ctx.fillStyle = '#010409';
        ctx.fillRect(0, 0, GAME_WIDTH, GAME_HEIGHT);

        // Draw entities
        entities.bullets.forEach(b => b.draw());
        entities.enemyBullets.forEach(b => b.draw());
        entities.enemies.forEach(e => e.draw());
        drawPlayer();
        drawFloatingText();
        
        // Draw Game Status
        if (game.gameOver) {
            ctx.fillStyle = 'rgba(0, 0, 0, 0.7)';
            ctx.fillRect(0, 0, GAME_WIDTH, GAME_HEIGHT);
            ctx.fillStyle = '#ff4444';
            ctx.font = "bold 48px 'Orbitron'";
            ctx.textAlign = 'center';
            ctx.fillText("GAME OVER", GAME_WIDTH / 2, GAME_HEIGHT / 2);
            ctx.font = "bold 24px 'Orbitron'";
            ctx.fillText(`Final Level: ${player.level} | Gold: ${player.gold}`, GAME_WIDTH / 2, GAME_HEIGHT / 2 + 50);
            ctx.font = "bold 18px 'Orbitron'";
            ctx.fillText("Press R to Restart", GAME_WIDTH / 2, GAME_HEIGHT / 2 + 100);
        } else if (game.isPaused) {
            ctx.fillStyle = 'rgba(0, 0, 0, 0.7)';
            ctx.fillRect(0, 0, GAME_WIDTH, GAME_HEIGHT);
            ctx.fillStyle = '#00ff99';
            ctx.font = "bold 48px 'Orbitron'";
            ctx.textAlign = 'center';
            ctx.fillText("PAUSED", GAME_WIDTH / 2, GAME_HEIGHT / 2);
        }
    }

    // Game loop using requestAnimationFrame
    function gameLoop() {
        update();
        draw();
        updateHUD();
        requestAnimationFrame(gameLoop);
    }

    // --- UPGRADE SHOP / MODAL LOGIC ---

    const upgradeModal = document.getElementById('upgradeModal');
    const upgradeButton = document.getElementById('upgradeButton');
    const closeModalButton = document.getElementById('closeModalButton');

    function toggleModal() {
        if (game.gameOver) return;

        game.isPaused = !game.isPaused;
        if (game.isPaused) {
            showModal();
        } else {
            hideModal();
        }
    }
    
    function showModal() {
        upgradeModal.classList.remove('hidden');
        updateModalStats();
    }

    function hideModal() {
        upgradeModal.classList.add('hidden');
    }

    function updateModalStats() {
        document.getElementById('modalSpValue').textContent = player.skillPoints;
        document.getElementById('currentDamage').textContent = player.damage;
        document.getElementById('currentMaxHealth').textContent = player.maxHealth;
        
        // Calculate the current cooldown time in ms
        const ms = player.fireRate.toFixed(0);
        document.getElementById('currentFireRate').textContent = `${ms}ms`;

        // Update button colors/availability
        document.querySelectorAll('.upgrade-item button').forEach(button => {
            const stat = button.getAttribute('data-stat');
            const spCost = parseInt(button.getAttribute('data-cost'));
            const goldCost = parseInt(button.getAttribute('data-cost'));
            
            let canAfford = false;
            let costType = 'SP';
            
            if (stat === 'heal') {
                costType = 'GOLD';
                canAfford = player.gold >= goldCost && player.health < player.maxHealth;
            } else {
                canAfford = player.skillPoints >= spCost;
            }

            button.disabled = !canAfford;
            button.classList.toggle('bg-gray-700', !canAfford);
            button.classList.toggle('bg-[#00ff99]', canAfford);
            button.textContent = `Cost: ${spCost} ${costType}`;
        });
    }

    function applyUpgrade(stat, cost, costType) {
        if (costType === 'SP') {
            if (player.skillPoints < cost) {
                createFloatingText("Not enough SP", GAME_WIDTH / 2, GAME_HEIGHT / 2, 40, '#ff0000');
                return;
            }
            player.skillPoints -= cost;
        } else if (costType === 'GOLD') {
            if (player.gold < cost) {
                createFloatingText("Not enough Gold", GAME_WIDTH / 2, GAME_HEIGHT / 2, 40, '#ff0000');
                return;
            }
            player.gold -= cost;
        }


        switch (stat) {
            case 'damage':
                player.damage += 1;
                createFloatingText(`DMG +1`, GAME_WIDTH / 2, GAME_HEIGHT / 2, 40, '#ff0000');
                break;
            case 'fireRate':
                // Increase fire rate by reducing cooldown (minimum 50ms)
                player.fireRate = Math.max(50, player.fireRate * 0.95); // 5% reduction
                createFloatingText(`FR +5%`, GAME_WIDTH / 2, GAME_HEIGHT / 2, 40, '#00ffff');
                break;
            case 'maxHealth':
                player.maxHealth += 10;
                player.health += 10; // Heal up to the new max health
                createFloatingText(`MAX HP +10`, GAME_WIDTH / 2, GAME_HEIGHT / 2, 40, '#00ff00');
                break;
            case 'heal':
                const healAmount = 50;
                player.health = Math.min(player.maxHealth, player.health + healAmount);
                createFloatingText(`HP +50`, GAME_WIDTH / 2, GAME_HEIGHT / 2, 40, '#00ff00');
                break;
        }

        updateModalStats();
        updateHUD();
    }
    
    // --- EVENT LISTENERS ---

    document.addEventListener('keydown', (e) => {
        if (game.gameOver && e.key === 'r') {
            initGame();
            return;
        }
        if (game.isPaused && e.key === 'u') {
             toggleModal();
             return;
        }
        if (!game.isRunning || game.gameOver || game.isPaused) return;

        if (e.key === 'ArrowLeft' || e.key === 'a') input.left = true;
        if (e.key === 'ArrowRight' || e.key === 'd') input.right = true;
        // Up Arrow now triggers shooting
        if (e.key === 'ArrowUp') input.shoot = true;
        if (e.key === 'u') toggleModal();
    });

    document.addEventListener('keyup', (e) => {
        if (e.key === 'ArrowLeft' || e.key === 'a') input.left = false;
        if (e.key === 'ArrowRight' || e.key === 'd') input.right = false;
        // Up Arrow now stops shooting
        if (e.key === 'ArrowUp') input.shoot = false;
    });

    upgradeButton.addEventListener('click', toggleModal);
    closeModalButton.addEventListener('click', toggleModal);
    
    document.getElementById('upgradesList').addEventListener('click', (e) => {
        if (e.target.tagName === 'BUTTON') {
            const button = e.target;
            const stat = button.getAttribute('data-stat');
            const cost = parseInt(button.getAttribute('data-cost'));
            let costType = button.textContent.includes('SP') ? 'SP' : 'GOLD';

            applyUpgrade(stat, cost, costType);
        }
    });

    // Removed the setupTouchControls function and the controls added by it.

    // --- INITIALIZATION ---

    window.onload = function () {
        initGame();
        // Removed setupTouchControls() call
        gameLoop();
    }
</script>
</body>
</html>
