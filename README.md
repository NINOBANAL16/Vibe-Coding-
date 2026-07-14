<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simple Shooting Game</title>
    <style>
        body {
            margin: 0;
            background: #111;
            color: white;
            font-family: sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            overflow: hidden;
        }
        canvas {
            border: 4px solid #333;
            background: #000;
            box-shadow: 0 0 20px rgba(0, 255, 0, 0.2);
        }
        #ui {
            margin-bottom: 10px;
            font-size: 20px;
        }
    </style>
</head>
<body>

    <div id="ui">Score: <span id="score">0</span> | Lives: <span id="lives">3</span></div>
    <canvas id="gameCanvas" width="600" height="500"></canvas>

<script>
const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");
const scoreEl = document.getElementById("score");
const livesEl = document.getElementById("lives");

// Game Variables
let score = 0;
let lives = 3;
let gameOver = false;

// Player Ship
const player = {
    x: canvas.width / 2 - 20,
    y: canvas.height - 40,
    width: 40,
    height: 20,
    speed: 6,
    color: "#00ff00"
};

// Controls
const keys = {};
document.addEventListener("keydown", (e) => keys[e.code] = true);
document.addEventListener("keyup", (e) => keys[e.code] = false);

// Lasers and Enemies
let lasers = [];
let enemies = [];
let enemySpawnTimer = 0;

// Shoot laser
document.addEventListener("keydown", (e) => {
    if (e.code === "Space" && !gameOver) {
        lasers.push({
            x: player.x + player.width / 2 - 3,
            y: player.y,
            width: 6,
            height: 15,
            speed: 8,
            color: "#ffff00"
        });
    }
    if (e.code === "KeyR" && gameOver) {
        resetGame();
    }
});

function spawnEnemy() {
    const size = 30;
    const x = Math.random() * (canvas.width - size);
    enemies.push({
        x: x,
        y: -size,
        width: size,
        height: size,
        speed: 2 + Math.random() * 2, // Random speed
        color: "#ff0055"
    });
}

function rectIntersect(rect1, rect2) {
    return rect1.x < rect2.x + rect2.width &&
           rect1.x + rect1.width > rect2.x &&
           rect1.y < rect2.y + rect2.height &&
           rect1.y + rect1.height > rect2.y;
}

function resetGame() {
    score = 0;
    lives = 3;
    lasers = [];
    enemies = [];
    gameOver = false;
    player.x = canvas.width / 2 - 20;
    scoreEl.innerText = score;
    livesEl.innerText = lives;
    gameLoop();
}

// Main Game Loop
function gameLoop() {
    if (gameOver) {
        ctx.fillStyle = "rgba(0, 0, 0, 0.75)";
        ctx.fillRect(0, 0, canvas.width, canvas.height);
        ctx.fillStyle = "white";
        ctx.font = "30px sans-serif";
        ctx.textAlign = "center";
        ctx.fillText("GAME OVER", canvas.width / 2, canvas.height / 2 - 10);
        ctx.font = "20px sans-serif";
        ctx.fillText("Press 'R' to Restart", canvas.width / 2, canvas.height / 2 + 30);
        return;
    }

    // Clear Canvas
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // Move Player
    if (keys["ArrowLeft"] && player.x > 0) player.x -= player.speed;
    if (keys["ArrowRight"] && player.x < canvas.width - player.width) player.x += player.speed;

    // Draw Player
    ctx.fillStyle = player.color;
    ctx.fillRect(player.x, player.y, player.width, player.height);

    // Handle Lasers
    for (let i = lasers.length - 1; i >= 0; i--) {
        let l = lasers[i];
        l.y -= l.speed;
        ctx.fillStyle = l.color;
        ctx.fillRect(l.x, l.y, l.width, l.height);

        // Remove off-screen lasers
        if (l.y < 0) lasers.splice(i, 1);
    }

    // Spawn Enemies
    enemySpawnTimer++;
    if (enemySpawnTimer % 40 === 0) { // Spawns every ~40 frames
        spawnEnemy();
    }

    // Handle Enemies
    for (let i = enemies.length - 1; i >= 0; i--) {
        let e = enemies[i];
        e.y += e.speed;
        ctx.fillStyle = e.color;
        ctx.fillRect(e.x, e.y, e.width, e.height);

        // Check laser collisions
        for (let j = lasers.length - 1; j >= 0; j--) {
            if (rectIntersect(lasers[j], e)) {
                enemies.splice(i, 1);
                lasers.splice(j, 1);
                score += 10;
                scoreEl.innerText = score;
                break; 
            }
        }

        // Check if enemy hits player or bottom floor
        if (enemies[i]) { // Check if it wasn't already destroyed by laser
            if (rectIntersect(e, player) || e.y > canvas.height) {
                enemies.splice(i, 1);
                lives--;
                livesEl.innerText = lives;
                if (lives <= 0) {
                    gameOver = true;
                }
            }
        }
    }

    requestAnimationFrame(gameLoop);
}

// Start the game
gameLoop();
</script>
</body>
</html>
