// Game configuration
const GAME_CONFIG = {
    CELL_SIZE: 40,
    PLAYER_SPEED: 4,
    ENEMY_SPEED_BASE: 2,
    ENEMY_SPEED_INCREMENT: 0.5,
    POWER_ORB_DURATION: 5000,
    HEARTBEAT_DISTANCE: 150,
    POWER_ORB_SCORE: 50,
    ORB_SCORE: 10,
    LEVEL_COMPLETE_SCORE: 100
};

// Audio context for sound effects
const AudioContext = window.AudioContext || window.webkitAudioContext;
let audioCtx = null;

// Game state
const gameState = {
    canvas: null,
    ctx: null,
    width: 0,
    height: 0,
    cellSize: GAME_CONFIG.CELL_SIZE,
    score: 0,
    level: 1,
    isPaused: false,
    isGameOver: false,
    isStarted: false,
    player: {
        x: 0,
        y: 0,
        radius: 15,
        speed: GAME_CONFIG.PLAYER_SPEED
    },
    enemies: [],
    orbs: [],
    powerOrbs: [],
    walls: [],
    exit: null,
    exitUnlocked: false,
    powerUpActive: false,
    powerUpEndTime: 0,
    heartbeatInterval: null
};

// Level definitions
const LEVELS = [
    {
        // Level 1 - Simple maze
        walls: [
            [0, 0, 20, 1], [0, 0, 1, 15], [19, 0, 1, 15], [0, 14, 20, 1],
            [5, 3, 10, 1], [5, 3, 1, 8], [14, 3, 1, 8], [5, 10, 10, 1],
            [8, 5, 1, 5], [11, 5, 1, 5], [3, 7, 14, 1]
        ],
        playerStart: [1, 1],
        enemyStart: [18, 13],
        exit: [18, 1],
        orbs: [
            [2, 2], [3, 2], [4, 2],
            [2, 3], [4, 3],
            [2, 4], [4, 4],
            [10, 2], [11, 2], [12, 2],
            [10, 3], [12, 3],
            [10, 4], [12, 4],
            [15, 7], [16, 7], [17, 7],
            [15, 8], [17, 8],
            [15, 9], [17, 9]
        ],
        powerOrbs: [[9, 7]]
    },
    {
        // Level 2 - Multiple exits
        walls: [
            [0, 0, 20, 1], [0, 0, 1, 15], [19, 0, 1, 15], [0, 14, 20, 1],
            [2, 2, 16, 1], [2, 2, 1, 12], [17, 2, 1, 12], [2, 13, 16, 1],
            [5, 5, 10, 1], [5, 5, 1, 8], [14, 5, 1, 8], [5, 12, 10, 1],
            [8, 8, 4, 1], [8, 8, 1, 4], [11, 8, 1, 4], [8, 11, 4, 1],
            [3, 7, 3, 1], [3, 7, 1, 3], [6, 7, 1, 3], [3, 9, 3, 1],
            [14, 7, 3, 1], [14, 7, 1, 3], [17, 7, 1, 3], [14, 9, 3, 1]
        ],
        playerStart: [1, 1],
        enemyStart: [18, 13],
        exit: [18, 1],
        fakeExits: [[1, 13], [18, 7]],
        orbs: [
            [3, 3], [4, 3], [5, 3],
            [3, 4], [5, 4],
            [3, 5], [5, 5],
            [8, 3], [9, 3], [10, 3], [11, 3],
            [8, 4], [11, 4],
            [8, 5], [11, 5],
            [14, 3], [15, 3], [16, 3],
            [14, 4], [16, 4],
            [14, 5], [16, 5],
            [3, 9], [4, 9], [5, 9],
            [3, 10], [5, 10],
            [3, 11], [5, 11],
            [14, 9], [15, 9], [16, 9],
            [14, 10], [16, 10],
            [14, 11], [16, 11]
        ],
        powerOrbs: [[9, 9], [7, 7], [12, 7]]
    },
    {
        // Level 3 - Killer is faster and smarter
        walls: [
            [0, 0, 20, 1], [0, 0, 1, 15], [19, 0, 1, 15], [0, 14, 20, 1],
            [3, 2, 14, 1], [3, 2, 1, 12], [16, 2, 1, 12], [3, 13, 14, 1],
            [6, 4, 8, 1], [6, 4, 1, 6], [13, 4, 1, 6], [6, 9, 8, 1],
            [9, 6, 2, 1], [9, 6, 1, 3], [11, 6, 1, 3], [9, 9, 2, 1],
            [3, 6, 2, 1], [3, 6, 1, 3], [5, 6, 1, 3], [3, 9, 2, 1],
            [15, 6, 2, 1], [15, 6, 1, 3], [17, 6, 1, 3], [15, 9, 2, 1],
            [8, 2, 1, 2], [11, 2, 1, 2], [8, 11, 1, 2], [11, 11, 1, 2]
        ],
        playerStart: [1, 1],
        enemyStart: [18, 13],
        exit: [18, 1],
        orbs: [
            [2, 2], [3, 2], [4, 2],
            [2, 3], [4, 3],
            [2, 4], [4, 4],
            [7, 2], [8, 2], [9, 2], [10, 2], [11, 2], [12, 2],
            [7, 3], [12, 3],
            [7, 4], [12, 4],
            [15, 2], [16, 2], [17, 2],
            [15, 3], [17, 3],
            [15, 4], [17, 4],
            [2, 11], [3, 11], [4, 11],
            [2, 12], [4, 12],
            [2, 13], [4, 13],
            [7, 11], [8, 11], [9, 11], [10, 11], [11, 11], [12, 11],
            [7, 12], [12, 12],
            [7, 13], [12, 13],
            [15, 11], [16, 11], [17, 11],
            [15, 12], [17, 12],
            [15, 13], [17, 13]
        ],
        powerOrbs: [[9, 7], [5, 7], [14, 7], [9, 5], [9, 9]]
    }
];

// Input handling
const keys = {};
document.addEventListener('keydown', (e) => {
    keys[e.key.toLowerCase()] = true;
    
    if (e.key === 'Escape') {
        togglePause();
    }
});

document.addEventListener('keyup', (e) => {
    keys[e.key.toLowerCase()] = false;
});

// Initialize audio on first user interaction
function initAudio() {
    if (!audioCtx) {
        audioCtx = new AudioContext();
    }
}

// Sound effects
function playSound(frequency, duration, type = 'sine', volume = 0.3) {
    if (!audioCtx) return;
    
    const oscillator = audioCtx.createOscillator();
    const gainNode = audioCtx.createGain();
    
    oscillator.connect(gainNode);
    gainNode.connect(audioCtx.destination);
    
    oscillator.type = type;
    oscillator.frequency.value = frequency;
    gainNode.gain.value = volume;
    
    oscillator.start();
    gainNode.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + duration);
    oscillator.stop(audioCtx.currentTime + duration);
}

function playOrbCollectSound() {
    playSound(800, 0.1, 'sine', 0.2);
    setTimeout(() => playSound(1000, 0.1, 'sine', 0.2), 50);
}

function playExitUnlockedSound() {
    playSound(523.25, 0.2, 'sine', 0.3);
    setTimeout(() => playSound(659.25, 0.2, 'sine', 0.3), 100);
    setTimeout(() => playSound(783.99, 0.3, 'sine', 0.3), 200);
}

function playGameOverSound() {
    playSound(110, 0.5, 'sawtooth', 0.4);
    setTimeout(() => playSound(82.41, 0.7, 'sawtooth', 0.4), 200);
}

function startHeartbeat() {
    if (gameState.heartbeatInterval) return;
    
    let beatCount = 0;
    gameState.heartbeatInterval = setInterval(() => {
        if (!gameState.isPaused && !gameState.isGameOver) {
            playSound(60 + (beatCount % 2) * 20, 0.1, 'triangle', 0.15);
            beatCount++;
        }
    }, 500);
}

function stopHeartbeat() {
    if (gameState.heartbeatInterval) {
        clearInterval(gameState.heartbeatInterval);
        gameState.heartbeatInterval = null;
    }
}

// Game initialization
function initGame() {
    gameState.canvas = document.getElementById('gameCanvas');
    gameState.ctx = gameState.canvas.getContext('2d');
    gameState.width = gameState.canvas.width;
    gameState.height = gameState.canvas.height;
    
    // Set up event listeners
    document.getElementById('startButton').addEventListener('click', startGame);
    document.getElementById('resumeButton').addEventListener('click', togglePause);
    document.getElementById('restartButton').addEventListener('click', restartLevel);
    document.getElementById('menuButton').addEventListener('click', showStartScreen);
    document.getElementById('playAgainButton').addEventListener('click', restartGame);
    document.getElementById('menuButton2').addEventListener('click', showStartScreen);
    
    // Start the game loop
    gameLoop();
}

function startGame() {
    initAudio(); // Initialize audio on first interaction
    document.getElementById('startScreen').style.display = 'none';
    gameState.isStarted = true;
    gameState.isGameOver = false;
    gameState.score = 0;
    gameState.level = 1;
    loadLevel(1);
}

function showStartScreen() {
    document.getElementById('startScreen').style.display = 'flex';
    document.getElementById('pauseMenu').style.display = 'none';
    document.getElementById('gameOver').style.display = 'none';
    gameState.isPaused = false;
    gameState.isGameOver = false;
    gameState.isStarted = false;
    stopHeartbeat();
}

function restartGame() {
    document.getElementById('gameOver').style.display = 'none';
    gameState.score = 0;
    gameState.level = 1;
    loadLevel(1);
}

function restartLevel() {
    document.getElementById('pauseMenu').style.display = 'none';
    gameState.isPaused = false;
    loadLevel(gameState.level);
}

function togglePause() {
    if (!gameState.isStarted || gameState.isGameOver) return;
    
    gameState.isPaused = !gameState.isPaused;
    document.getElementById('pauseMenu').style.display = gameState.isPaused ? 'block' : 'none';
}

function loadLevel(levelNum) {
    const level = LEVELS[levelNum - 1];
    if (!level) {
        alert(`Congratulations! You've completed all levels with a score of ${gameState.score}!`);
        showStartScreen();
        return;
    }
    
    // Reset game state
    gameState.walls = [];
    gameState.orbs = [];
    gameState.powerOrbs = [];
    gameState.enemies = [];
    gameState.exitUnlocked = false;
    gameState.powerUpActive = false;
    gameState.powerUpEndTime = 0;
    
    // Load walls
    level.walls.forEach(wall => {
        gameState.walls.push({
            x: wall[0] * gameState.cellSize,
            y: wall[1] * gameState.cellSize,
            width: wall[2] * gameState.cellSize,
            height: wall[3] * gameState.cellSize
        });
    });
    
    // Load player
    gameState.player.x = level.playerStart[0] * gameState.cellSize + gameState.cellSize / 2;
    gameState.player.y = level.playerStart[1] * gameState.cellSize + gameState.cellSize / 2;
    
    // Load enemy
    const enemy = {
        x: level.enemyStart[0] * gameState.cellSize + gameState.cellSize / 2,
        y: level.enemyStart[1] * gameState.cellSize + gameState.cellSize / 2,
        radius: 15,
        speed: GAME_CONFIG.ENEMY_SPEED_BASE + (levelNum - 1) * GAME_CONFIG.ENEMY_SPEED_INCREMENT,
        smartAI: levelNum >= 3
    };
    gameState.enemies.push(enemy);
    
    // Load exit
    gameState.exit = {
        x: level.exit[0] * gameState.cellSize,
        y: level.exit[1] * gameState.cellSize,
        width: gameState.cellSize,
        height: gameState.cellSize
    };
    
    // Load orbs
    level.orbs.forEach(orb => {
        gameState.orbs.push({
            x: orb[0] * gameState.cellSize + gameState.cellSize / 2,
            y: orb[1] * gameState.cellSize + gameState.cellSize / 2,
            radius: 8,
            collected: false
        });
    });
    
    // Load power orbs
    level.powerOrbs.forEach(powerOrb => {
        gameState.powerOrbs.push({
            x: powerOrb[0] * gameState.cellSize + gameState.cellSize / 2,
            y: powerOrb[1] * gameState.cellSize + gameState.cellSize / 2,
            radius: 12,
            collected: false
        });
    });
    
    // Load fake exits for level 2
    if (level.fakeExits) {
        level.fakeExits.forEach(fakeExit => {
            gameState.walls.push({
                x: fakeExit[0] * gameState.cellSize,
                y: fakeExit[1] * gameState.cellSize,
                width: gameState.cellSize,
                height: gameState.cellSize,
                isFakeExit: true
            });
        });
    }
    
    // Update UI
    document.getElementById('levelValue').textContent = levelNum;
    updateScore();
}

function updateScore() {
    document.getElementById('scoreValue').textContent = gameState.score;
}

function checkCollision(rect1, rect2) {
    return rect1.x < rect2.x + rect2.width &&
           rect1.x + rect1.width > rect2.x &&
           rect1.y < rect2.y + rect2.height &&
           rect1.y + rect1.height > rect2.y;
}

function checkCircleCollision(circle1, circle2) {
    const dx = circle1.x - circle2.x;
    const dy = circle1.y - circle2.y;
    const distance = Math.sqrt(dx * dx + dy * dy);
    return distance < circle1.radius + circle2.radius;
}

function updatePlayer() {
    if (gameState.isPaused || gameState.isGameOver) return;
    
    const player = gameState.player;
    let dx = 0;
    let dy = 0;
    
    if (keys['w'] || keys['arrowup']) dy = -player.speed;
    if (keys['s'] || keys['arrowdown']) dy = player.speed;
    if (keys['a'] || keys['arrowleft']) dx = -player.speed;
    if (keys['d'] || keys['arrowright']) dx = player.speed;
    
    // Normalize diagonal movement
    if (dx !== 0 && dy !== 0) {
        dx *= 0.707;
        dy *= 0.707;
    }
    
    const playerRect = {
        x: player.x - player.radius,
        y: player.y - player.radius,
        width: player.radius * 2,
        height: player.radius * 2
    };
    
    const newPlayerRect = {
        x: player.x + dx - player.radius,
        y: player.y + dy - player.radius,
        width: player.radius * 2,
        height: player.radius * 2
    };
    
    // Check wall collisions
    let canMove = true;
    for (const wall of gameState.walls) {
        if (checkCollision(newPlayerRect, wall)) {
            canMove = false;
            break;
        }
    }
    
    // Check fake exit collisions
    for (const wall of gameState.walls) {
        if (wall.isFakeExit && checkCollision(newPlayerRect, wall)) {
            canMove = false;
            break;
        }
    }
    
    if (canMove) {
        player.x += dx;
        player.y += dy;
        
        // Keep player within bounds
        player.x = Math.max(player.radius, Math.min(gameState.width - player.radius, player.x));
        player.y = Math.max(player.radius, Math.min(gameState.height - player.radius, player.y));
    }
    
    // Check orb collection
    gameState.orbs.forEach((orb, index) => {
        if (!orb.collected && checkCircleCollision(player, orb)) {
            orb.collected = true;
            gameState.score += GAME_CONFIG.ORB_SCORE;
            updateScore();
            playOrbCollectSound();
            
            if (gameState.orbs.every(o => o.collected)) {
                gameState.exitUnlocked = true;
                playExitUnlockedSound();
            }
        }
    });
    
    // Check power orb collection
    gameState.powerOrbs.forEach((powerOrb, index) => {
        if (!powerOrb.collected && checkCircleCollision(player, powerOrb)) {
            powerOrb.collected = true;
            gameState.score += GAME_CONFIG.POWER_ORB_SCORE;
            updateScore();
            playOrbCollectSound();
            
            gameState.powerUpActive = true;
            gameState.powerUpEndTime = Date.now() + GAME_CONFIG.POWER_ORB_DURATION;
        }
    });
    
    // Check exit collision
    if (gameState.exitUnlocked && checkCollision(playerRect, gameState.exit)) {
        gameState.score += GAME_CONFIG.LEVEL_COMPLETE_SCORE;
        updateScore();
        
        setTimeout(() => {
            loadLevel(gameState.level + 1);
        }, 1000);
    }
}

function updateEnemies() {
    if (gameState.isPaused || gameState.isGameOver) return;
    
    const player = gameState.player;
    
    gameState.enemies.forEach(enemy => {
        // Check if power-up is active
        if (gameState.powerUpActive && Date.now() < gameState.powerUpEndTime) {
            return;
        }
        
        let dx = player.x - enemy.x;
        let dy = player.y - enemy.y;
        const distance = Math.sqrt(dx * dx + dy * dy);
        
        dx /= distance;
        dy /= distance;
        
        // Smart AI for level 3
        if (enemy.smartAI) {
            const playerSpeed = gameState.player.speed;
            const futureX = player.x + (keys['d'] ? playerSpeed : keys['a'] ? -playerSpeed : 0);
            const futureY = player.y + (keys['s'] ? playerSpeed : keys['w'] ? -playerSpeed : 0);
            
            const futureDx = futureX - enemy.x;
            const futureDy = futureY - enemy.y;
            const futureDistance = Math.sqrt(futureDx * futureDx + futureDy * futureDy);
            
            if (futureDistance < distance) {
                dx = futureDx / futureDistance;
                dy = futureDy / futureDistance;
            }
        }
        
        const enemyRect = {
            x: enemy.x - enemy.radius,
            y: enemy.y - enemy.radius,
            width: enemy.radius * 2,
            height: enemy.radius * 2
        };
        
        const newEnemyRect = {
            x: enemy.x + dx * enemy.speed - enemy.radius,
            y: enemy.y + dy * enemy.speed - enemy.radius,
            width: enemy.radius * 2,
            height: enemy.radius * 2
        };
        
        // Check wall collisions
        let canMove = true;
        for (const wall of gameState.walls) {
            if (checkCollision(newEnemyRect, wall)) {
                canMove = false;
                break;
            }
        }
        
        if (canMove) {
            enemy.x += dx * enemy.speed;
            enemy.y += dy * enemy.speed;
        }
        
        // Check collision with player
        if (checkCircleCollision(player, enemy)) {
            gameOver();
        }
        
        // Start heartbeat when enemy is close
        if (distance < GAME_CONFIG.HEARTBEAT_DISTANCE && !gameState.heartbeatInterval) {
            startHeartbeat();
        } else if (distance >= GAME_CONFIG.HEARTBEAT_DISTANCE && gameState.heartbeatInterval) {
            stopHeartbeat();
        }
    });
    
    // Check if power-up has expired
    if (gameState.powerUpActive && Date.now() >= gameState.powerUpEndTime) {
        gameState.powerUpActive = false;
    }
}

function gameOver() {
    gameState.isGameOver = true;
    playGameOverSound();
    stopHeartbeat();
    
    document.getElementById('finalScore').textContent = gameState.score;
    document.getElementById('gameOver').style.display = 'block';
}

function render() {
    const ctx = gameState.ctx;
    
    // Clear canvas
    ctx.fillStyle = '#0a0a0a';
    ctx.fillRect(0, 0, gameState.width, gameState.height);
    
    // Draw walls with neon glow
    ctx.shadowColor = '#00aaff';
    ctx.shadowBlur = 15;
    ctx.fillStyle = '#0066cc';
    
    gameState.walls.forEach(wall => {
        if (!wall.isFakeExit) {
            ctx.fillRect(wall.x, wall.y, wall.width, wall.height);
        }
    });
    
    // Draw fake exits
    ctx.shadowColor = '#ff00ff';
    ctx.fillStyle = '#9900cc';
    
    gameState.walls.forEach(wall => {
        if (wall.isFakeExit) {
            ctx.fillRect(wall.x, wall.y, wall.width, wall.height);
        }
    });
    
    // Draw exit (only when unlocked)
    if (gameState.exitUnlocked) {
        ctx.shadowColor = '#00ff00';
        ctx.shadowBlur = 20;
        ctx.fillStyle = '#00cc00';
        ctx.fillRect(gameState.exit.x, gameState.exit.y, gameState.exit.width, gameState.exit.height);
        
        ctx.shadowBlur = 0;
        ctx.fillStyle = '#ffffff';
        ctx.font = '20px Arial';
        ctx.textAlign = 'center';
        ctx.textBaseline = 'middle';
        ctx.fillText('EXIT', gameState.exit.x + gameState.exit.width/2, gameState.exit.y + gameState.exit.height/2);
    }
    
    // Draw orbs with pulsing effect
    ctx.shadowBlur = 10;
    
    gameState.orbs.forEach(orb => {
        if (!orb.collected) {
            const pulse = Math.sin(Date.now() * 0.005) * 0.2 + 0.8;
            ctx.shadowColor = '#ffff00';
            ctx.fillStyle = `rgba(255, 255, 0, ${pulse})`;
            ctx.beginPath();
            ctx.arc(orb.x, orb.y, orb.radius, 0, Math.PI * 2);
            ctx.fill();
        }
    });
    
    // Draw power orbs
    gameState.powerOrbs.forEach(powerOrb => {
        if (!powerOrb.collected) {
            const pulse = Math.sin(Date.now() * 0.003) * 0.3 + 0.7;
            ctx.shadowColor = '#ff00ff';
            ctx.fillStyle = `rgba(255, 0, 255, ${pulse})`;
            ctx.beginPath();
            ctx.arc(powerOrb.x, powerOrb.y, powerOrb.radius, 0, Math.PI * 2);
            ctx.fill();
            
            // Draw star shape
            ctx.shadowBlur = 0;
            ctx.fillStyle = '#ffffff';
            ctx.beginPath();
            for (let i = 0; i < 5; i++) {
                const angle = (i * 2 * Math.PI / 5) - Math.PI / 2;
                const x = powerOrb.x + Math.cos(angle) * (powerOrb.radius * 0.6);
                const y = powerOrb.y + Math.sin(angle) * (powerOrb.radius * 0.6);
                
                if (i === 0) {
                    ctx.moveTo(x, y);
                } else {
                    ctx.lineTo(x, y);
                }
                
                const innerAngle = angle + Math.PI / 5;
                const innerX = powerOrb.x + Math.cos(innerAngle) * (powerOrb.radius * 0.3);
                const innerY = powerOrb.y + Math.sin(innerAngle) * (powerOrb.radius * 0.3);
                ctx.lineTo(innerX, innerY);
            }
            ctx.closePath();
            ctx.fill();
        }
    });
    
    // Draw player
    ctx.shadowColor = '#00ffff';
    ctx.shadowBlur = 15;
    ctx.fillStyle = '#00ffff';
    ctx.beginPath();
    ctx.arc(gameState.player.x, gameState.player.y, gameState.player.radius, 0, Math.PI * 2);
    ctx.fill();
    
    // Draw player direction indicator
    ctx.shadowBlur = 0;
    ctx.fillStyle = '#ffffff';
    ctx.beginPath();
    ctx.arc(gameState.player.x, gameState.player.y, gameState.player.radius * 0.4, 0, Math.PI * 2);
    ctx.fill();
    
    // Draw enemies
    gameState.enemies.forEach(enemy => {
        if (gameState.powerUpActive && Date.now() < gameState.powerUpEndTime) {
            ctx.shadowColor = '#00ffff';
            ctx.shadowBlur = 20;
            ctx.fillStyle = '#88ddff';
        } else {
            ctx.shadowColor = '#ff0000';
            ctx.shadowBlur = 15;
            ctx.fillStyle = '#ff0000';
        }
        
        ctx.beginPath();
        ctx.arc(enemy.x, enemy.y, enemy.radius, 0, Math.PI * 2);
        ctx.fill();
        
        // Draw enemy eyes
        ctx.shadowBlur = 0;
        ctx.fillStyle = '#ffffff';
        
        ctx.beginPath();
        ctx.arc(enemy.x - enemy.radius * 0.3, enemy.y - enemy.radius * 0.2, enemy.radius * 0.2, 0, Math.PI * 2);
        ctx.fill();
        
        ctx.beginPath();
        ctx.arc(enemy.x + enemy.radius * 0.3, enemy.y - enemy.radius * 0.2, enemy.radius * 0.2, 0, Math.PI * 2);
        ctx.fill();
        
        // Draw enemy pupils
        const angleToPlayer = Math.atan2(gameState.player.y - enemy.y, gameState.player.x - enemy.x);
        const pupilOffset = enemy.radius * 0.1;
        
        ctx.fillStyle = '#000000';
        
        ctx.beginPath();
        ctx.arc(
            enemy.x - enemy.radius * 0.3 + Math.cos(angleToPlayer) * pupilOffset,
            enemy.y - enemy.radius * 0.2 + Math.sin(angleToPlayer) * pupilOffset,
            enemy.radius * 0.1,
            0,
            Math.PI * 2
        );
        ctx.fill();
        
        ctx.beginPath();
        ctx.arc(
            enemy.x + enemy.radius * 0.3 + Math.cos(angleToPlayer) * pupilOffset,
            enemy.y - enemy.radius * 0.2 + Math.sin(angleToPlayer) * pupilOffset,
            enemy.radius * 0.1,
            0,
            Math.PI * 2
        );
        ctx.fill();
    });
    
    // Reset shadow
    ctx.shadowBlur = 0;
    
    // Draw power-up indicator
    if (gameState.powerUpActive) {
        const timeLeft = (gameState.powerUpEndTime - Date.now()) / 1000;
        const barWidth = 200;
        const barHeight = 10;
        const barX = (gameState.width - barWidth) / 2;
        const barY = 50;
        
        ctx.fillStyle = 'rgba(0, 0, 0, 0.5)';
        ctx.fillRect(barX, barY, barWidth, barHeight);
        
        ctx.fillStyle = '#ff00ff';
        ctx.fillRect(barX, barY, barWidth * (timeLeft / 5), barHeight);
        
        ctx.strokeStyle = '#ffffff';
        ctx.lineWidth = 1;
        ctx.strokeRect(barX, barY, barWidth, barHeight);
        
        ctx.fillStyle = '#ffffff';
        ctx.font = '14px Arial';
        ctx.textAlign = 'center';
        ctx.fillText(`POWER-UP: ${timeLeft.toFixed(1)}s`, gameState.width / 2, barY - 5);
    }
}

function gameLoop() {
    updatePlayer();
    updateEnemies();
    render();
    
    requestAnimationFrame(gameLoop);
}

// Initialize game when page loads
window.addEventListener('load', initGame);
body {
    margin: 0;
    padding: 0;
    background-color: #0a0a0a;
    color: #fff;
    font-family: 'Courier New', monospace;
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
    overflow: hidden;
}

#gameContainer {
    position: relative;
    box-shadow: 0 0 30px rgba(0, 150, 255, 0.5);
    border-radius: 10px;
    overflow: hidden;
}

#gameCanvas {
    background-color: #111;
    display: block;
}

#gameUI {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    padding: 15px;
    display: flex;
    justify-content: space-between;
    pointer-events: none;
}

.score {
    font-size: 18px;
    color: #0ff;
    text-shadow: 0 0 10px rgba(0, 255, 255, 0.7);
}

.level {
    font-size: 18px;
    color: #f0f;
    text-shadow: 0 0 10px rgba(255, 0, 255, 0.7);
}

#pauseMenu {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    background-color: rgba(0, 0, 0, 0.9);
    border: 2px solid #0ff;
    border-radius: 10px;
    padding: 30px;
    display: none;
    text-align: center;
    box-shadow: 0 0 30px rgba(0, 150, 255, 0.8);
}

#pauseMenu h2 {
    margin-top: 0;
    color: #f0f;
    text-shadow: 0 0 15px rgba(255, 0, 255, 0.8);
}

#pauseMenu button {
    background-color: #0ff;
    color: #000;
    border: none;
    padding: 10px 20px;
    margin: 10px;
    border-radius: 5px;
    cursor: pointer;
    font-family: inherit;
    font-weight: bold;
    transition: all 0.3s;
}

#pauseMenu button:hover {
    background-color: #fff;
    box-shadow: 0 0 15px rgba(0, 255, 255, 0.8);
    transform: scale(1.05);
}

#gameOver {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    background-color: rgba(0, 0, 0, 0.9);
    border: 2px solid #f00;
    border-radius: 10px;
    padding: 30px;
    display: none;
    text-align: center;
    box-shadow: 0 0 30px rgba(255, 0, 0, 0.8);
}

#gameOver h2 {
    margin-top: 0;
    color: #f00;
    text-shadow: 0 0 15px rgba(255, 0, 0, 0.8);
}

#instructions {
    position: absolute;
    bottom: 10px;
    left: 50%;
    transform: translateX(-50%);
    text-align: center;
    font-size: 14px;
    color: #888;
}

#startScreen {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: rgba(0, 0, 0, 0.95);
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    z-index: 100;
}

#startScreen h1 {
    font-size: 48px;
    margin-bottom: 20px;
    color: #f0f;
    text-shadow: 0 0 20px rgba(255, 0, 255, 0.8);
    animation: pulse 2s infinite;
}

@keyframes pulse {
    0% { transform: scale(1); }
    50% { transform: scale(1.05); }
    100% { transform: scale(1); }
}

#startButton {
    background-color: #0ff;
    color: #000;
    border: none;
    padding: 15px 30px;
    border-radius: 5px;
    cursor: pointer;
    font-family: inherit;
    font-weight: bold;
    font-size: 20px;
    transition: all 0.3s;
}

#startButton:hover {
    background-color: #fff;
    box-shadow: 0 0 20px rgba(0, 255, 255, 0.8);
    transform: scale(1.05);
}
