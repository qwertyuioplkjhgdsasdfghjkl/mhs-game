<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Space Shooter Game</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            background: url('blue.png') no-repeat center center fixed;
            background-size: cover;
        }

        #gameCanvas {
            display: none;
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
        }

        #logo {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: black;
            display: flex;
            justify-content: center;
            align-items: center;
            animation: fadeOut 4s forwards;
        }

        #logo img {
            width: 100vw;
            height: 100vh;
            object-fit: cover;
            transform: scale(0);
            animation: zoomIn 4s forwards;
        }

        @keyframes zoomIn {
            0% {
                transform: scale(0);
                opacity: 0;
            }
            100% {
                transform: scale(1);
                opacity: 1;
            }
        }

        @keyframes fadeOut {
            0% {
                opacity: 1;
            }
            100% {
                opacity: 0;
                display: none;
            }
        }

        #levelIndicator {
            position: absolute;
            top: 10px;
            left: 50%;
            transform: translateX(-50%);
            font-size: 24px;
            color: white;
            background: rgba(0, 0, 0, 0.5);
            padding: 10px 20px;
            border-radius: 5px;
        }

        #gameOver {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            text-align: center;
            background: rgba(0, 0, 0, 0.7);
            color: white;
            padding: 20px;
            border-radius: 10px;
            font-size: 24px;
            display: none;
        }
    </style>
</head>
<body>

<div id="logo">
    <img src="logo.jpeg" alt="Logo">
</div>

<div id="levelIndicator">Level: 1</div>
<canvas id="gameCanvas"></canvas>

<div id="gameOver">Game Over
    <button id="retryButton">Retry</button>
</div>

<script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const logo = document.getElementById('logo');
    const levelIndicator = document.getElementById('levelIndicator');
    const gameOverDiv = document.getElementById('gameOver');
    const retryButton = document.getElementById('retryButton');

    let spaceHero = { x: window.innerWidth / 2, y: window.innerHeight - 60, width: 50, height: 50, speed: 15 };
    let enemies = [];
    let bullets = [];
    let gameInterval, bulletInterval, level = 1;
    let isGameOver = false;
    let touchStartX = 0, touchEndX = 0;

    function startGame() {
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;
        logo.style.display = 'block';

        setTimeout(() => {
            logo.style.display = 'none';
            canvas.style.display = 'block';
            levelIndicator.style.display = 'block';
            gameOverDiv.style.display = 'none';
            isGameOver = false;
            level = 1;
            enemies = [];
            bullets = [];
            updateLevelIndicator();
            spawnEnemies();
            gameInterval = setInterval(gameLoop, 1000 / 60);
            bulletInterval = setInterval(fireBullet, 300); // Automatically fire bullets every 300ms
        }, 4000);
    }

    function fireBullet() {
        if (!isGameOver) {
            bullets.push({ x: spaceHero.x + spaceHero.width / 2 - 2, y: spaceHero.y, width: 4, height: 10 });
        }
    }

    function gameLoop() {
        if (isGameOver) return;
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        drawDangerLine();
        updateGameObjects();
        drawGameObjects();
        checkCollisions();
        checkLevelUp();
    }

    function drawDangerLine() {
        const dangerLineY = spaceHero.y - 50; // 1 inch above the hero
        ctx.strokeStyle = 'red';
        ctx.lineWidth = 2;
        ctx.beginPath();
        ctx.moveTo(0, dangerLineY);
        ctx.lineTo(canvas.width, dangerLineY);
        ctx.stroke();

        enemies.forEach(enemy => {
            if (enemy.y + enemy.height >= dangerLineY) {
                gameOver();
            }
        });
    }

    function updateGameObjects() {
        bullets.forEach(bullet => bullet.y -= 5);
        bullets = bullets.filter(bullet => bullet.y > 0);

        enemies.forEach(enemy => {
            enemy.y += 2;
            if (enemy.y > canvas.height) {
                enemy.y = -50;
                enemy.x = Math.random() * (canvas.width - 50);
            }
        });
    }

    function drawGameObjects() {
        const heroImg = new Image();
        heroImg.src = 'hero.png';
        ctx.drawImage(heroImg, spaceHero.x, spaceHero.y, spaceHero.width, spaceHero.height);

        enemies.forEach(enemy => {
            const enemyImg = new Image();
            enemyImg.src = 'enemy1.png';
            ctx.drawImage(enemyImg, enemy.x, enemy.y, enemy.width, enemy.height);
        });

        ctx.fillStyle = 'yellow';
        bullets.forEach(bullet => ctx.fillRect(bullet.x, bullet.y, bullet.width, bullet.height));
    }

    function checkCollisions() {
        bullets.forEach((bullet, bulletIndex) => {
            enemies.forEach((enemy, enemyIndex) => {
                if (
                    bullet.x < enemy.x + enemy.width &&
                    bullet.x + bullet.width > enemy.x &&
                    bullet.y < enemy.y + enemy.height &&
                    bullet.y + bullet.height > enemy.y
                ) {
                    bullets.splice(bulletIndex, 1);
                    enemies.splice(enemyIndex, 1);
                }
            });
        });

        enemies.forEach(enemy => {
            if (
                spaceHero.x < enemy.x + enemy.width &&
                spaceHero.x + spaceHero.width > enemy.x &&
                spaceHero.y < enemy.y + enemy.height &&
                spaceHero.y + spaceHero.height > enemy.y
            ) {
                gameOver();
            }
        });
    }

    function checkLevelUp() {
        if (enemies.length === 0) {
            level++;
            updateLevelIndicator();
            spawnEnemies();
        }
    }

    function spawnEnemies() {
        for (let i = 0; i < level; i++) {
            enemies.push({
                x: Math.random() * (canvas.width - 50),
                y: Math.random() * -500,
                width: 50,
                height: 50
            });
        }
    }

    function updateLevelIndicator() {
        levelIndicator.textContent = `Level: ${level}`;
    }

    function gameOver() {
        clearInterval(gameInterval);
        clearInterval(bulletInterval);
        gameOverDiv.style.display = 'block';
        isGameOver = true;
    }

    retryButton.addEventListener('click', () => {
        gameOverDiv.style.display = 'none'; // Immediately hide game over UI
        startGame();
    });

    document.addEventListener('keydown', (event) => {
        if (isGameOver) return;
        if (event.key === 'ArrowLeft' && spaceHero.x > 0) {
            spaceHero.x -= spaceHero.speed;
        }
        if (event.key === 'ArrowRight' && spaceHero.x < canvas.width - spaceHero.width) {
            spaceHero.x += spaceHero.speed;
        }
    });

    document.addEventListener('touchstart', (event) => {
        touchStartX = event.touches[0].clientX;
    });

    document.addEventListener('touchend', (event) => {
        touchEndX = event.changedTouches[0].clientX;
        handleSwipe();
    });

    function handleSwipe() {
        const swipeDistance = touchEndX - touchStartX;
        const minSwipeDistance = 50; // Minimum distance to detect swipe

        if (swipeDistance > minSwipeDistance && spaceHero.x < canvas.width - spaceHero.width) {
            // Swipe Right
            spaceHero.x += spaceHero.speed;
        } else if (swipeDistance < -minSwipeDistance && spaceHero.x > 0) {
            // Swipe Left
            spaceHero.x -= spaceHero.speed;
        }
    }

    startGame();
</script>
</body>
</html>
