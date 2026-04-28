<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Midnight Bed Battle: Fixed Physics</title>
    <style>
        body { margin: 0; overflow: hidden; background: #0a0a1a; font-family: 'Arial', sans-serif; touch-action: none; }
        canvas { display: block; }
        #ui { position: absolute; top: 10px; width: 100%; text-align: center; color: white; pointer-events: none; }
        #meter-bg { width: 200px; height: 12px; background: #222; border: 2px solid #fff; margin: 8px auto; border-radius: 6px; overflow: hidden; }
        #meter-fill { height: 100%; width: 0%; background: #0f0; transition: width 0.1s; }
        .btn {
            position: absolute; bottom: 30px; width: 90px; height: 90px;
            background: rgba(255, 255, 255, 0.2); border: 4px solid #fff;
            border-radius: 50%; color: white; font-size: 40px;
            display: flex; align-items: center; justify-content: center;
            user-select: none; cursor: pointer; pointer-events: auto;
        }
        #lBtn { left: 25px; }
        #rBtn { right: 25px; }
        .btn:active { background: rgba(255,255,255,0.5); transform: scale(0.9); }
    </style>
</head>
<body>
    <div id="ui">
        <h2 id="msg">SHHH! BE QUIET!</h2>
        <div id="meter-bg"><div id="meter-fill"></div></div>
        <div>SCORE: <span id="score">0</span></div>
    </div>
    <div id="lBtn" class="btn">◀</div>
    <div id="rBtn" class="btn">▶</div>
    <canvas id="game"></canvas>

<script>
    const canvas = document.getElementById('game');
    const ctx = canvas.getContext('2d');
    const meterFill = document.getElementById('meter-fill');
    const scoreEl = document.getElementById('score');

    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;

    // CONFIG
    const BED_W = Math.min(canvas.width * 0.85, 550);
    const CENTER = canvas.width / 2;
    let score = 0, tilt = 0, gameActive = true, yell = 0, frame = 0;

    const child = { x: 0, vx: 0, speed: 0.65, fric: 0.93 };
    const pL = { x: -BED_W/2.5, target: -BED_W/2.5, restless: false };
    const pR = { x: BED_W/2.5, target: BED_W/2.5, restless: false };
    let teddy = { x: 0, active: false };

    // INPUT
    const keys = { l: false, r: false };
    const listen = (id, k) => {
        const b = document.getElementById(id);
        b.addEventListener('pointerdown', () => keys[k] = true);
        b.addEventListener('pointerup', () => keys[k] = false);
    };
    listen('lBtn', 'l'); listen('rBtn', 'r');

    function drawStick(x, y, scale, color, isChild) {
        ctx.save();
        ctx.translate(x, y);
        let bob = Math.sin(frame * 0.1) * 3;
        ctx.strokeStyle = color; ctx.lineWidth = isChild ? 4 : 8; ctx.lineCap = 'round';
        // Head
        ctx.beginPath(); ctx.arc(0, -50*scale + bob, 15*scale, 0, 7); ctx.stroke();
        // Torso
        ctx.beginPath(); ctx.moveTo(0, -35*scale + bob); ctx.lineTo(0, 15*scale); ctx.stroke();
        // Arms
        let w = !isChild ? Math.sin(frame*0.05)*15 : 0;
        ctx.beginPath(); ctx.moveTo(-25*scale, -15*scale + w); ctx.lineTo(25*scale, -15*scale - w); ctx.stroke();
        // Legs
        ctx.beginPath(); ctx.moveTo(0, 15*scale); ctx.lineTo(-18*scale, 45*scale);
        ctx.moveTo(0, 15*scale); ctx.lineTo(18*scale, 45*scale); ctx.stroke();
        ctx.restore();
    }

    function update() {
        if (!gameActive) return;
        frame++;

        // Yell Meter
        if (keys.l || keys.r) yell = Math.min(100, yell + 1.2);
        else yell = Math.max(0, yell - 0.4);
        meterFill.style.width = yell + "%";
        meterFill.style.background = yell > 75 ? '#f00' : '#0f0';

        // Parents AI - Restless logic
        [pL, pR].forEach((p, i) => {
            if (yell > 85) p.restless = true;
            if (p.restless) {
                p.target = i === 0 ? -40 : 40; // Charge middle
                if (yell < 15) p.restless = false;
            } else if (Math.abs(p.x - p.target) < 10) {
                let side = i === 0 ? -1 : 1;
                p.target = side * (50 + Math.random() * (BED_W/2 - 70));
            }
            p.x += (p.target - p.x) * (p.restless ? 0.04 : 0.012);
        });

        // TILT PHYSICS (Fixed: Now tilts toward the net weight)
        // If pL is at -200 and pR is at 100, net weight is -100 (tilt left)
        tilt = (pL.x + pR.x) / BED_W * 0.4;

        if (keys.l) child.vx -= child.speed;
        if (keys.r) child.vx += child.speed;
        child.vx += tilt * 3.5; // Gravity pull
        child.vx *= child.fric;
        child.x += child.vx;

        // Teddy Logic
        if (!teddy.active && Math.random() < 0.005) {
            teddy.x = (Math.random() - 0.5) * BED_W * 0.8;
            teddy.active = true;
        }
        if (teddy.active && Math.abs(child.x - teddy.x) < 30) {
            teddy.active = false; yell = Math.max(0, yell - 30);
        }

        // Death Checks
        if (Math.abs(child.x - pL.x) < 35 || Math.abs(child.x - pR.x) < 35) die("SQUISHED!");
        if (Math.abs(child.x) > BED_W/2) die("OFF THE BED!");

        score += 0.05; scoreEl.innerText = Math.floor(score);
        draw();
        requestAnimationFrame(update);
    }

    function die(m) { gameActive = false; alert(m + "\nScore: " + Math.floor(score)); location.reload(); }

    function draw() {
        ctx.fillStyle = '#050510'; ctx.fillRect(0, 0, canvas.width, canvas.height); // Floor

        ctx.save();
        ctx.translate(CENTER, canvas.height * 0.65);
        ctx.rotate(tilt);

        // Bed (Mattress and Pillows)
        ctx.fillStyle = '#321'; ctx.fillRect(-BED_W/2 - 10, 0, BED_W + 20, 50); // Frame
        ctx.fillStyle = '#eee'; ctx.fillRect(-BED_W/2, -15, BED_W, 20); // Mattress
        ctx.fillStyle = '#fff'; // Pillows
        ctx.fillRect(-BED_W/2 + 20, -35, 70, 20); ctx.fillRect(BED_W/2 - 90, -35, 70, 20);

        // Draw Teddy
        if (teddy.active) {
            ctx.fillStyle = '#853'; ctx.beginPath();
            ctx.arc(teddy.x, -10, 12, 0, 7); ctx.fill(); // Teddy Head
        }

        drawStick(pL.x, -25, 1.1, '#47f', false); // Parent Left (Blue Pajamas)
        drawStick(pR.x, -25, 1.1, '#f47', false); // Parent Right (Pink Pajamas)
        drawStick(child.x, -10, 0.6, '#0f0', true); // Child (Green Pajamas)

        ctx.restore();
    }
    update();
</script>
</body>
</html>
