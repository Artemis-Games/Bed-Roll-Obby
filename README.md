<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Midnight Bed Battle: Stick Figure Edition</title>
    <style>
        body { margin: 0; overflow: hidden; background-color: #222; font-family: 'Courier New', Courier, monospace; }
        canvas { display: block; }
        #ui { position: absolute; top: 20px; width: 100%; text-align: center; color: white; pointer-events: none; text-shadow: 2px 2px #000; }
    </style>
</head>
<body>
    <div id="ui">
        <h1>DON'T GET SQUISHED!</h1>
        <p>Use LEFT/RIGHT Arrows to stay on the bed.</p>
        <h2>Score: <span id="score">0</span></h2>
    </div>
    <canvas id="gameCanvas"></canvas>

<script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const scoreEl = document.getElementById('score');

    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;

    const BED_WIDTH = 500;
    const BED_X = (canvas.width - BED_WIDTH) / 2;
    let score = 0;
    let gameActive = true;

    const keys = { ArrowLeft: false, ArrowRight: false };
    window.addEventListener('keydown', e => keys[e.key] = true);
    window.addEventListener('keyup', e => keys[e.key] = false);

    const player = { x: canvas.width / 2, y: canvas.height * 0.7, speed: 6, width: 30 };
    
    // Parent logic: x is current position, targetX is where they want to roll to
    const leftParent = { x: BED_X + 50, targetX: BED_X + 50, side: 'left' };
    const rightParent = { x: BED_X + BED_WIDTH - 50, targetX: BED_X + BED_WIDTH - 50, side: 'right' };

    function drawStickFigure(x, y, scale, isChild, color) {
        ctx.strokeStyle = color;
        ctx.lineWidth = isChild ? 3 : 5;
        ctx.beginPath();

        // Head
        ctx.arc(x, y - (40 * scale), 15 * scale, 0, Math.PI * 2);
        // Body
        ctx.moveTo(x, y - (25 * scale));
        ctx.lineTo(x, y + (10 * scale));
        // Arms
        ctx.moveTo(x - (20 * scale), y - (10 * scale));
        ctx.lineTo(x + (20 * scale), y - (10 * scale));
        // Legs
        ctx.moveTo(x, y + (10 * scale));
        ctx.lineTo(x - (15 * scale), y + (40 * scale));
        ctx.moveTo(x, y + (10 * scale));
        ctx.lineTo(x + (15 * scale), y + (40 * scale));
        
        ctx.stroke();
    }

    function gameOver() {
        gameActive = false;
        alert("You fell or got squished! Final Score: " + Math.floor(score));
        document.location.reload();
    }

    function update() {
        if (!gameActive) return;

        // Player Movement
        if (keys.ArrowLeft) player.x -= player.speed;
        if (keys.ArrowRight) player.x += player.speed;

        // Parent Rolling Logic
        [leftParent, rightParent].forEach(p => {
            if (Math.abs(p.x - p.targetX) < 5) {
                // Pick a new spot to roll to every few seconds
                let range = 120;
                if (p.side === 'left') {
                    p.targetX = BED_X + (Math.random() * range);
                } else {
                    p.targetX = (BED_X + BED_WIDTH) - (Math.random() * range);
                }
            }
            // Move toward target
            p.x += (p.targetX - p.x) * 0.02;
        });

        // Collision Checks
        if (player.x < leftParent.x + 20 || player.x > rightParent.x - 20) gameOver(); // Squished
        if (player.x < BED_X || player.x > BED_X + BED_WIDTH) gameOver(); // Lava

        score += 0.05;
        scoreEl.innerText = Math.floor(score);

        draw();
        requestAnimationFrame(update);
    }

    function draw() {
        // Lava
        ctx.fillStyle = '#cc3300';
        ctx.fillRect(0, 0, canvas.width, canvas.height);
        
        // Bed
        ctx.fillStyle = '#fff';
        ctx.fillRect(BED_X, 0, BED_WIDTH, canvas.height);
        
        // Bed Sheets pattern
        ctx.strokeStyle = '#ddd';
        ctx.lineWidth = 1;
        for(let i = 0; i < canvas.height; i += 20) {
            ctx.beginPath();
            ctx.moveTo(BED_X, i);
            ctx.lineTo(BED_X + BED_WIDTH, i);
            ctx.stroke();
        }

        // Draw Parents (Large stick figures)
        drawStickFigure(leftParent.x, canvas.height * 0.7, 1.2, false, '#444');
        drawStickFigure(rightParent.x, canvas.height * 0.7, 1.2, false, '#444');

        // Draw Child (Small stick figure)
        drawStickFigure(player.x, player.y, 0.6, true, '#000');
    }

    update();
</script>
</body>
</html>
