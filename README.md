<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Midnight Seesaw: Pro Edition</title>
    <style>
        body { margin: 0; overflow: hidden; background-color: #1a0a00; font-family: 'Segoe UI', sans-serif; touch-action: none; }
        canvas { display: block; }
        #ui { position: absolute; top: 10px; width: 100%; text-align: center; color: white; pointer-events: none; }
        #yellMeter { width: 200px; height: 15px; background: #333; border: 2px solid #fff; margin: 10px auto; border-radius: 10px; overflow: hidden; }
        #yellFill { height: 100%; width: 0%; background: #ff0; transition: width 0.1s; }
        .control-btn {
            position: absolute; bottom: 40px; width: 100px; height: 100px;
            background: rgba(255, 255, 255, 0.15); border: 4px solid rgba(255, 255, 255, 0.6);
            border-radius: 20px; color: white; font-size: 50px;
            display: flex; align-items: center; justify-content: center;
            user-select: none; pointer-events: auto; backdrop-filter: blur(5px);
        }
        #leftBtn { left: 40px; }
        #rightBtn { right: 40px; }
        .control-btn:active { background: rgba(255, 255, 255, 0.4); transform: scale(0.9); }
    </style>
</head>
<body>
    <div id="ui">
        <h2 id="status">SHHH... DON'T WAKE THEM!</h2>
        <div id="yellMeter"><div id="yellFill"></div></div>
        <h3>Score: <span id="score">0</span></h3>
    </div>

    <div id="leftBtn" class="control-btn">◀</div>
    <div id="rightBtn" class="control-btn">▶</div>

    <canvas id="gameCanvas"></canvas>

<script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const scoreEl = document.getElementById('score');
    const yellFill = document.getElementById('yellFill');
    const statusText = document.getElementById('status');

    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;

    const BED_WIDTH = Math.min(window.innerWidth * 0.85, 600);
    const CENTER_X = canvas.width / 2;
    let score = 0, tilt = 0, gameActive = true, yellLevel = 0, frame = 0;

    const player = { x: CENTER_X, vx: 0, speed: 0.7, friction: 0.92, anim: 0 };
    const pLeft = { x: -BED_WIDTH/3, targetX: -BED_WIDTH/3, side: 'L', restless: false };
    const pRight = { x: BED_WIDTH/3, targetX: BED_WIDTH/3, side: 'R', restless: false };

    const keys = { left: false, right: false };
    const setupInput = (id, key) => {
        const btn = document.getElementById(id);
        btn.addEventListener('pointerdown', () => keys[key] = true);
        btn.addEventListener('pointerup', () => keys[key] = false);
    };
    setupInput('leftBtn', 'left'); setupInput('rightBtn', 'right');

    function drawStickPerson(x, y, scale, color, isChild) {
        ctx.save();
        ctx.translate(x, y);
        let bounce = Math.sin(frame * 0.1) * 2;
        ctx.strokeStyle = color; ctx.lineWidth = isChild ? 4 : 7;
        ctx.lineCap = 'round';

        // Head
        ctx.beginPath(); ctx.arc(0, -50*scale + bounce, 18*scale, 0, Math.PI*2); ctx.stroke();
        // Body
        ctx.beginPath(); ctx.moveTo(0, -32*scale + bounce); ctx.lineTo(0, 15*scale); ctx.stroke();
        // Arms (rolling animation for parents)
        let armWiggle = !isChild ? Math.sin(frame * 0.05) * 10 : 0;
        ctx.beginPath(); ctx.moveTo(-25*scale, -15*scale + armWiggle); ctx.lineTo(25*scale, -15*scale - armWiggle); ctx.stroke();
        // Legs
        ctx.beginPath(); ctx.moveTo(0, 15*scale); ctx.lineTo(-20*scale, 50*scale);
        ctx.moveTo(0, 15*scale); ctx.lineTo(20*scale, 50*scale); ctx.stroke();
        ctx.restore();
    }

    function update() {
        if (!gameActive) return;
        frame++;

        // 1. Yell Meter Logic
        if (keys.left || keys.right) yellLevel = Math.min(100, yellLevel + 1.5);
        else yellLevel = Math.max(0, yellLevel - 0.5);
        yellFill.style.width = yellLevel + "%";
        yellFill.style.background = yellLevel > 70 ? '#f00' : '#ff0';

        // 2. Parent AI
        [pLeft, pRight].forEach(p => {
            if (yellLevel > 80) { p.restless = true; statusText.innerText = "THEY'RE WAKING UP!"; }
            if (p.restless) {
                p.targetX = p.side === 'L' ? -50 : 50; // Charge center
                if (yellLevel < 20) p.restless = false;
            } else if (Math.abs(p.x - p.targetX) < 10) {
                p.targetX = p.side === 'L' ? -(100 + Math.random() * (BED_WIDTH/2 - 100)) : (100 + Math.random() * (BED_WIDTH/2 - 100));
            }
            p.x += (p.targetX - p.x) * (p.restless ? 0.05 : 0.015);
        });

        // 3. Physics & Tilt
        tilt = (pRight.x + pLeft.x) / (BED_WIDTH) * 0.3;
        if (keys.left) player.vx -= player.speed;
        if (keys.right) player.vx += player.speed;
        player.vx += tilt * 3; player.vx *= player.friction; player.x += player.vx;

        // 4. Death
        if (Math.abs(player.x - pLeft.x) < 40 || Math.abs(player.x - pRight.x) < 40) end("SQUISHED!");
        if (Math.abs(player.x) > BED_WIDTH/2) end("LAVA!");

        score += 0.05; scoreEl.innerText = Math.floor(score);
        draw();
        requestAnimationFrame(update);
    }

    function end(msg) { gameActive = false; alert(msg + " Score: " + Math.floor(score)); location.reload(); }

    function draw() {
        ctx.fillStyle = '#200'; ctx.fillRect(0, 0, canvas.width, canvas.height); // Dark floor

        ctx.save();
        ctx.translate(CENTER_X, canvas.height * 0.65);
        ctx.rotate(tilt);

        // Bed Frame & Mattress
        ctx.fillStyle = '#4a2c13'; ctx.fillRect(-BED_WIDTH/2 - 10, 0, BED_WIDTH + 20, 40); // Wood
        ctx.fillStyle = '#eee'; ctx.fillRect(-BED_WIDTH/2, -15, BED_WIDTH, 20); // Mattress
        
        // Pillows
        ctx.fillStyle = '#fff';
        ctx.fillRect(-BED_WIDTH/2 + 20, -30, 80, 20);
        ctx.fillRect(BED_WIDTH/2 - 100, -30, 80, 20);

        // Headboard
        ctx.fillStyle = '#3a1c03'; ctx.fillRect(-BED_WIDTH/2 - 10, -80, 15, 80);
        ctx.fillRect(BED_WIDTH/2 - 5, -80, 15, 80);

        drawStickPerson(pLeft.x, -25, 1.1, '#333', false);
        drawStickPerson(pRight.x, -25, 1.1, '#333', false);
        drawStickPerson(player.x, -10, 0.6, '#000', true);

        ctx.restore();
    }
    update();
</script>
</body>
</html>
