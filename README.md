<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Midnight Bed Battle</title>
    <style>
        body { margin: 0; overflow: hidden; background-color: #333; color: white; font-family: sans-serif; }
        canvas { display: block; background: #555; }
        #ui { position: absolute; top: 10px; left: 10px; pointer-events: none; }
    </style>
</head>
<body>
    <div id="ui">
        <h1>Midnight Bed Battle</h1>
        <p>Use LEFT/RIGHT arrows to move. Avoid parents & lava!</p>
        <p>Score: <span id="score">0</span></p>
    </div>
    <canvas id="gameCanvas"></canvas>

<script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const scoreEl = document.getElementById('score');

    // Resize canvas
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;

    // Game Constants
    const BED_WIDTH = 400;
    const BED_HEIGHT = canvas.height;
    const BED_X = (canvas.width - BED_WIDTH) / 2;
    const PLAYER_SIZE = 30;
    const PARENT_WIDTH = 60;
    const PARENT_HEIGHT = 100;
    const LAVA_COLOR = '#ff4500';

    let score = 0;
    let gameActive = true;

    // Game Objects
    const player = {
        x: canvas.width / 2,
        y: canvas.height - 100,
        size: PLAYER_SIZE,
        speed: 5
    };

    const parentLeft = { x: BED_X - 10, y: 0, w: PARENT_WIDTH, h: PARENT_HEIGHT, speed: 2 };
    const parentRight = { x: BED_X + BED_WIDTH - PARENT_WIDTH + 10, y: 0, w: PARENT_WIDTH, h: PARENT_HEIGHT, speed: 2 };

    // Input Handling
    const keys = { ArrowLeft: false, ArrowRight: false };
    window.addEventListener('keydown', e => keys[e.key] = true);
    window.addEventListener('keyup', e => keys[e.key] = false);

    function gameOver() {
        gameActive = false;
        alert("Game Over! You fell in the lava! Score: " + Math.floor(score));
        document.location.reload();
    }

    function update() {
        if (!gameActive) return;

        // Move Player
        if (keys.ArrowLeft) player.x -= player.speed;
        if (keys.ArrowRight) player.x += player.speed;

        // Parents move randomly back and forth
        parentLeft.x += (Math.random() - 0.5) * 5;
        parentRight.x += (Math.random() - 0.5) * 5;

        // Constrain Parents to bed edges
        parentLeft.x = Math.max(BED_X - 50, Math.min(parentLeft.x, BED_X + 50));
        parentRight.x = Math.max(BED_X + BED_WIDTH - 50, Math.min(parentRight.x, BED_X + BED_WIDTH + 10));

        // Collision: Parent Squish
        if (player.x < parentLeft.x + parentLeft.w || player.x + player.size > parentRight.x) {
            gameOver();
        }

        // Collision: Bed Edges (Lava)
        if (player.x < BED_X || player.x + player.size > BED_X + BED_WIDTH) {
            gameOver();
        }

        // Update Score
        score += 0.1;
        scoreEl.innerText = Math.floor(score);

        draw();
        requestAnimationFrame(update);
    }

    function draw() {
        // Draw Lava Background
        ctx.fillStyle = LAVA_COLOR;
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        // Draw Bed
        ctx.fillStyle = '#f0e68c';
        ctx.fillRect(BED_X, 0, BED_WIDTH, BED_HEIGHT);

        // Draw Parents
        ctx.fillStyle = '#8b4513'; // Brown (Parents)
        ctx.fillRect(parentLeft.x, parentLeft.y, parentLeft.w, parentLeft.h);
        ctx.fillRect(parentRight.x, parentRight.y, parentRight.w, parentRight.h);

        // Draw Player (Child)
        ctx.fillStyle = '#32cd32'; // Green (Child)
        ctx.fillRect(player.x, player.y, player.size, player.size);
    }

    update();
</script>
</body>
</html>
