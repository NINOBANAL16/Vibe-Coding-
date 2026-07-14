WE VIBE CODING       ctx.font = "20px sans-serif";
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
