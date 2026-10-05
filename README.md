# mm-game
MM
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MM Footsies Engine</title>
    <style>
        body {
            background-color: #111;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            font-family: monospace, sans-serif;
            color: #fff;
        }
        canvas {
            border: 4px solid #fff;
            background-color: #222;
        }
        #controls {
            margin-top: 15px;
            text-align: center;
            font-size: 14px;
            line-height: 1.6;
        }
    </style>
</head>
<body>

<canvas id="gameCanvas" width="800" height="400"></canvas>
<div id="controls">
    <span style="color:#3498db">■ P1 (Djrey):</span> A/D = Walk | W = Jump | F = Poke Attack<br>
    <span style="color:#e74c3c">■ P2 (Cobolt):</span> Left/Right = Walk | Up = Jump | L = Poke Attack
</div>

<script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');

    const gravity = 0.6;
    const groundY = 280;

    const keys = {
        a: false, d: false, w: false, f: false,
        ArrowLeft: false, ArrowRight: false, ArrowUp: false, l: false
    };

    const player1 = {
        x: 150, y: groundY, vx: 0, vy: 0, width: 40, height: 80,
        color: '#3498db', isGrounded: true, isAttacking: false, attackTimer: 0,
        attackBox: { x: 0, y: 0, width: 50, height: 20 }
    };

    const player2 = {
        x: 610, y: groundY, vx: 0, vy: 0, width: 40, height: 80,
        color: '#e74c3c', isGrounded: true, isAttacking: false, attackTimer: 0,
        attackBox: { x: 0, y: 0, width: 50, height: 20 }
    };

    window.addEventListener('keydown', (e) => {
        if (e.key in keys) keys[e.key] = true;
        if (e.key.toLowerCase() in keys) keys[e.key.toLowerCase()] = true;
    });

    window.addEventListener('keyup', (e) => {
        if (e.key in keys) keys[e.key] = false;
        if (e.key.toLowerCase() in keys) keys[e.key.toLowerCase()] = false;
    });

    function checkCollision(rect1, rect2) {
        return (
            rect1.x < rect2.x + rect2.width &&
            rect1.x + rect1.width > rect2.x &&
            rect1.y < rect2.y + rect2.height &&
            rect1.y + rect1.height > rect2.y
        );
    }

    function updatePlayer(p, moveLeft, moveRight, jump, attack, facingRight) {
        if (moveLeft && p.x > 0) p.vx = -5;
        else if (moveRight && p.x + p.width < canvas.width) p.vx = 5;
        else p.vx = 0;

        p.x += p.vx;

        if (jump && p.isGrounded) {
            p.vy = -12;
            p.isGrounded = false;
        }

        p.vy += gravity;
        p.y += p.vy;

        if (p.y >= groundY) {
            p.y = groundY;
            p.vy = 0;
            p.isGrounded = true;
        }

        if (attack && !p.isAttacking) {
            p.isAttacking = true;
            p.attackTimer = 12;
        }

        if (p.isAttacking) {
            p.attackTimer--;
            p.attackBox.x = facingRight ? (p.x + p.width) : (p.x - p.attackBox.width);
            p.attackBox.y = p.y + 20;

            if (p.attackTimer <= 0) p.isAttacking = false;
        }
    }

    function gameLoop() {
        const p1FacingRight = player1.x < player2.x;
        const p2FacingRight = player2.x < player1.x;

        updatePlayer(player1, keys.a, keys.d, keys.w, keys.f, p1FacingRight);
        updatePlayer(player2, keys.ArrowLeft, keys.ArrowRight, keys.ArrowUp, keys.l, p2FacingRight);

        const p1HitP2 = player1.isAttacking && checkCollision(player1.attackBox, player2);
        const p2HitP1 = player2.isAttacking && checkCollision(player2.attackBox, player1);

        ctx.clearRect(0, 0, canvas.width, canvas.height);

        ctx.fillStyle = '#555';
        ctx.fillRect(0, 360, canvas.width, 40);

        ctx.fillStyle = p2HitP1 ? '#ffffff' : player1.color;
        ctx.fillRect(player1.x, player1.y, player1.width, player1.height);

        if (player1.isAttacking) {
            ctx.fillStyle = '#f1c40f';
            ctx.fillRect(player1.attackBox.x, player1.attackBox.y, player1.attackBox.width, player1.attackBox.height);
        }

        ctx.fillStyle = p1HitP2 ? '#ffffff' : player2.color;
        ctx.fillRect(player2.x, player2.y, player2.width, player2.height);

        if (player2.isAttacking) {
            ctx.fillStyle = '#f1c40f';
            ctx.fillRect(player2.attackBox.x, player2.attackBox.y, player2.attackBox.width, player2.attackBox.height);
        }

        requestAnimationFrame(gameLoop);
    }

    gameLoop();
</script>

</body>
</html>
