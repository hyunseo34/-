# -html_code = """<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>웹 마리오 스타일 게임 (Super Mario Style Web Game)</title>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
        }

        body {
            background-color: #1a1a2e;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            color: #ffffff;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            padding: 20px;
        }

        header {
            text-align: center;
            margin-bottom: 15px;
        }

        h1 {
            font-size: 2rem;
            color: #e94560;
            text-shadow: 2px 2px #0f3460;
            margin-bottom: 5px;
        }

        p.subtitle {
            font-size: 0.95rem;
            color: #8f94fb;
        }

        #game-container {
            position: relative;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
            border-radius: 12px;
            overflow: hidden;
            border: 4px solid #0f3460;
            background: #5c94fc;
        }

        canvas {
            display: block;
            background: linear-gradient(to bottom, #6b8cff, #a2b9ff 70%, #c4d4ff);
        }

        .controls-info {
            margin-top: 15px;
            background: #16213e;
            padding: 12px 25px;
            border-radius: 8px;
            border: 1px solid #0f3460;
            display: flex;
            gap: 20px;
            font-size: 0.9rem;
            color: #dcdde1;
        }

        .key {
            background: #0f3460;
            padding: 2px 8px;
            border-radius: 4px;
            border: 1px solid #4e54c8;
            font-weight: bold;
            color: #e94560;
        }

        .guide-box {
            margin-top: 25px;
            max-width: 800px;
            width: 100%;
            background: #16213e;
            border-radius: 10px;
            padding: 20px;
            border: 1px solid #0f3460;
        }

        .guide-box h2 {
            font-size: 1.2rem;
            color: #e94560;
            margin-bottom: 10px;
            border-bottom: 2px solid #0f3460;
            padding-bottom: 5px;
        }

        .guide-box ol {
            padding-left: 20px;
            line-height: 1.6;
            color: #c8d6e5;
            font-size: 0.95rem;
        }

        .guide-box code {
            background: #0f3460;
            padding: 2px 6px;
            border-radius: 4px;
            color: #4cd137;
            font-family: monospace;
        }
    </style>
</head>
<body>

    <header>
        <h1>SUPER MARIO STYLE GAME</h1>
        <p class="subtitle">HTML5 Canvas & JavaScript로 구현한 2D 플랫포머 게임</p>
    </header>

    <div id="game-container">
        <canvas id="gameCanvas" width="800" height="450"></canvas>
    </div>

    <div class="controls-info">
        <div>조작법:</div>
        <div><span class="key">←</span> / <span class="key">→</span> 또는 <span class="key">A</span> / <span class="key">D</span> : 이동</div>
        <div><span class="key">Space</span> 또는 <span class="key">↑</span> / <span class="key">W</span> : 점프</div>
        <div><span class="key">R</span> : 재시작</div>
    </div>

    <div class="guide-box">
        <h2>🚀 GitHub Pages로 무료 배포하는 방법</h2>
        <ol>
            <li>GitHub에 로그인 후 새 저장소(Repository)를 만듭니다 (예: <code>mario-game</code>).</li>
            <li>이 파일의 이름을 <code>index.html</code>로 지정하여 해당 저장소에 업로드(Upload files)합니다.</li>
            <li>저장소의 <strong>Settings</strong> 탭으로 이동합니다.</li>
            <li>좌측 메뉴에서 <strong>Pages</strong>를 클릭합니다.</li>
            <li><strong>Build and deployment</strong>의 Branch 섹션에서 <code>main</code> (또는 <code>master</code>)을 선택하고 <strong>Save</strong>를 누릅니다.</li>
            <li>1~2분 후 생성되는 URL(예: <code>https://사용자명.github.io/mario-game/</code>)로 접속하면 완성된 게임을 바로 즐길 수 있습니다!</li>
        </ol>
    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');

        // 게임 기본 상태
        let score = 0;
        let lives = 3;
        let gameOver = false;
        let gameWin = false;

        // 물리 변수
        const gravity = 0.5;
        const friction = 0.8;

        // 키 입력 관리
        const keys = {
            right: false,
            left: false,
            up: false
        };

        // 카메라/화면 스크롤 X Offset
        let cameraX = 0;
        const levelWidth = 3200; // 전체 맵 길이

        // 플레이어 (마리오)
        const player = {
            x: 100,
            y: 300,
            width: 30,
            height: 40,
            velocityX: 0,
            velocityY: 0,
            speed: 5,
            jumpPower: -12,
            grounded: false,
            facing: 'right',
            reset: function() {
                this.x = 100;
                this.y = 300;
                this.velocityX = 0;
                this.velocityY = 0;
                this.grounded = false;
            }
        };

        // 맵 요소: 블록, 발판, 깃대
        const platforms = [
            // 바닥 (중간중간 구멍 설치)
            { x: 0, y: 400, width: 800, height: 50, type: 'ground' },
            { x: 900, y: 400, width: 700, height: 50, type: 'ground' },
            { x: 1700, y: 400, width: 800, height: 50, type: 'ground' },
            { x: 2600, y: 400, width: 600, height: 50, type: 'ground' },

            // 공중 발판 및 블록
            { x: 250, y: 280, width: 40, height: 40, type: 'itemBlock', hit: false },
            { x: 290, y: 280, width: 120, height: 40, type: 'brick' },
            { x: 330, y: 180, width: 40, height: 40, type: 'itemBlock', hit: false },

            { x: 600, y: 300, width: 80, height: 20, type: 'platform' },
            { x: 750, y: 220, width: 80, height: 20, type: 'platform' },

            { x: 1050, y: 280, width: 160, height: 40, type: 'brick' },
            { x: 1130, y: 180, width: 40, height: 40, type: 'itemBlock', hit: false },

            // 파이프 (장애물/발판)
            { x: 500, y: 320, width: 60, height: 80, type: 'pipe' },
            { x: 1350, y: 300, width: 60, height: 100, type: 'pipe' },
            { x: 1900, y: 320, width: 60, height: 80, type: 'pipe' },

            // 계단 구조
            { x: 2200, y: 360, width: 40, height: 40, type: 'brick' },
            { x: 2240, y: 320, width: 40, height: 80, type: 'brick' },
            { x: 2280, y: 280, width: 40, height: 120, type: 'brick' },
            { x: 2320, y: 240, width: 40, height: 160, type: 'brick' },

            // 골인 지점 깃대
            { x: 3000, y: 150, width: 10, height: 250, type: 'flagpole' },
            { x: 2985, y: 150, width: 40, height: 30, type: 'flag' }
        ];

        // 코인 목록
        let coins = [
            { x: 260, y: 240, collected: false },
            { x: 620, y: 260, collected: false },
            { x: 650, y: 260, collected: false },
            { x: 770, y: 180, collected: false },
            { x: 1070, y: 240, collected: false },
            { x: 1110, y: 240, collected: false },
            { x: 1150, y: 240, collected: false },
            { x: 1750, y: 350, collected: false },
            { x: 1800, y: 350, collected: false }
        ];

        // 적 (굼바 스타일)
        let enemies = [
            { x: 450, y: 370, width: 30, height: 30, vx: -1.5, alive: true, minX: 350, maxX: 490 },
            { x: 980, y: 370, width: 30, height: 30, vx: -1.5, alive: true, minX: 910, maxX: 1200 },
            { x: 1500, y: 370, width: 30, height: 30, vx: -2, alive: true, minX: 1420, maxX: 1650 },
            { x: 2000, y: 370, width: 30, height: 30, vx: -1.5, alive: true, minX: 1960, maxX: 2150 }
        ];

        // 키 이벤트 리스너
        window.addEventListener('keydown', (e) => {
            if (e.key === 'ArrowRight' || e.key === 'd' || e.key === 'D') keys.right = true;
            if (e.key === 'ArrowLeft' || e.key === 'a' || e.key === 'A') keys.left = true;
            if ((e.key === 'ArrowUp' || e.key === 'w' || e.key === 'W' || e.key === ' ') && player.grounded) {
                player.velocityY = player.jumpPower;
                player.grounded = false;
            }
            if (e.key === 'r' || e.key === 'R') {
                restartGame();
            }
        });

        window.addEventListener('keyup', (e) => {
            if (e.key === 'ArrowRight' || e.key === 'd' || e.key === 'D') keys.right = false;
            if (e.key === 'ArrowLeft' || e.key === 'a' || e.key === 'A') keys.left = false;
        });

        function restartGame() {
            score = 0;
            lives = 3;
            gameOver = false;
            gameWin = false;
            player.reset();
            
            // 코인, 적, 아이템블록 복구
            coins.forEach(c => c.collected = false);
            enemies.forEach(e => e.alive = true);
            platforms.forEach(p => {
                if(p.type === 'itemBlock') p.hit = false;
            });
        }

        // 업데이트 로직
        function update() {
            if (gameOver || gameWin) return;

            // 이동 처리
            if (keys.right) {
                if (player.velocityX < player.speed) player.velocityX++;
                player.facing = 'right';
            }
            if (keys.left) {
                if (player.velocityX > -player.speed) player.velocityX--;
                player.facing = 'left';
            }

            // 마찰력 및 중력
            player.velocityX *= friction;
            player.velocityY += gravity;

            player.grounded = false;

            // X축 위치 업데이트 및 충돌 검사
            player.x += player.velocityX;
            checkHorizontalCollisions();

            // Y축 위치 업데이트 및 충돌 검사
            player.y += player.velocityY;
            checkVerticalCollisions();

            // 카메라 위치 설정 (플레이어 중심 추적)
            cameraX = player.x - canvas.width / 3;
            if (cameraX < 0) cameraX = 0;
            if (cameraX > levelWidth - canvas.width) cameraX = levelWidth - canvas.width;

            // 코인 획득 검사
            coins.forEach(coin => {
                if (!coin.collected &&
                    player.x < coin.x + 20 &&
                    player.x + player.width > coin.x &&
                    player.y < coin.y + 20 &&
                    player.y + player.height > coin.y) {
                    coin.collected = true;
                    score += 100;
                }
            });

            // 적 업데이트 및 충돌
            enemies.forEach(enemy => {
                if (!enemy.alive) return;

                enemy.x += enemy.vx;
                if (enemy.x <= enemy.minX || enemy.x + enemy.width >= enemy.maxX) {
                    enemy.vx *= -1;
                }

                // 플레이어와 적 충돌 판정
                if (player.x < enemy.x + enemy.width &&
                    player.x + player.width > enemy.x &&
                    player.y < enemy.y + enemy.height &&
                    player.y + player.height > enemy.y) {

                    // 위에서 밟은 경우
                    if (player.velocityY > 0 && player.y + player.height - player.velocityY <= enemy.y + 10) {
                        enemy.alive = false;
                        player.velocityY = player.jumpPower * 0.7; // 밟고 튕겨오르기
                        score += 200;
                    } else {
                        // 옆이나 아래에서 닿아 피격당함
                        playerHit();
                    }
                }
            });

            // 낭떠러지 추락 검사
            if (player.y > canvas.height + 100) {
                playerHit();
            }

            // 승리 조건 (깃대에 도달)
            if (player.x >= 3000) {
                gameWin = true;
            }
        }

        function checkHorizontalCollisions() {
            platforms.forEach(p => {
                if (p.type === 'flag' || p.type === 'flagpole') return;
                if (rectIntersect(player.x, player.y, player.width, player.height, p.x, p.y, p.width, p.height)) {
                    if (player.velocityX > 0) {
                        player.x = p.x - player.width;
                    } else if (player.velocityX < 0) {
                        player.x = p.x + p.width;
                    }
                }
            });
        }

        function checkVerticalCollisions() {
            platforms.forEach(p => {
                if (p.type === 'flag' || p.type === 'flagpole') return;
                if (rectIntersect(player.x, player.y, player.width, player.height, p.x, p.y, p.width, p.height)) {
                    if (player.velocityY > 0) { // 아래로 낙하 중 (발판을 밟음)
                        player.y = p.y - player.height;
                        player.velocityY = 0;
                        player.grounded = true;
                    } else if (player.velocityY < 0) { // 위로 상승 중 (블록 아래 머리 부딪힘)
                        player.y = p.y + p.height;
                        player.velocityY = 0;

                        // 물음표 아이템 블록 타격
                        if (p.type === 'itemBlock' && !p.hit) {
                            p.hit = true;
                            score += 250;
                        }
                    }
                }
            });
        }

        function playerHit() {
            lives--;
            if (lives <= 0) {
                gameOver = true;
            } else {
                player.reset();
            }
        }

        function rectIntersect(x1, y1, w1, h1, x2, y2, w2, h2) {
            return x1 < x2 + w2 && x1 + w1 > x2 && y1 < y2 + h2 && y1 + h1 > y2;
        }

        // 그리기 (Draw)
        function draw() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            ctx.save();
            // 카메라 스크롤 적용
            ctx.translate(-cameraX, 0);

            // 배경 구름 및 산 표현
            drawBackgroundDetails();

            // 플랫폼 / 블록 그리기
            platforms.forEach(p => {
                if (p.type === 'ground') {
                    ctx.fillStyle = '#8b4513';
                    ctx.fillRect(p.x, p.y, p.width, p.height);
                    ctx.fillStyle = '#2ed573';
                    ctx.fillRect(p.x, p.y, p.width, 10);
                } else if (p.type === 'brick') {
                    ctx.fillStyle = '#cd6133';
                    ctx.fillRect(p.x, p.y, p.width, p.height);
                    ctx.strokeStyle = '#5c2305';
                    ctx.strokeRect(p.x, p.y, p.width, p.height);
                } else if (p.type === 'itemBlock') {
                    ctx.fillStyle = p.hit ? '#8a8a8a' : '#f1c40f';
                    ctx.fillRect(p.x, p.y, p.width, p.height);
                    ctx.strokeStyle = '#d35400';
                    ctx.strokeRect(p.x, p.y, p.width, p.height);

                    if (!p.hit) {
                        ctx.fillStyle = '#d35400';
                        ctx.font = 'bold 20px sans-serif';
                        ctx.fillText('?', p.x + 13, p.y + 27);
                    }
                } else if (p.type === 'pipe') {
                    ctx.fillStyle = '#2ed573';
                    ctx.fillRect(p.x, p.y, p.width, p.height);
                    ctx.strokeStyle = '#26af5f';
                    ctx.strokeRect(p.x, p.y, p.width, p.height);
                    // 파이프 캡
                    ctx.fillRect(p.x - 4, p.y, p.width + 8, 20);
                    ctx.strokeRect(p.x - 4, p.y, p.width + 8, 20);
                } else if (p.type === 'platform') {
                    ctx.fillStyle = '#e1b12c';
                    ctx.fillRect(p.x, p.y, p.width, p.height);
                } else if (p.type === 'flagpole') {
                    ctx.fillStyle = '#dcdde1';
                    ctx.fillRect(p.x, p.y, p.width, p.height);
                } else if (p.type === 'flag') {
                    ctx.fillStyle = '#e84118';
                    ctx.beginPath();
                    ctx.moveTo(p.x, p.y);
                    ctx.lineTo(p.x + p.width, p.y + 15);
                    ctx.lineTo(p.x, p.y + 30);
                    ctx.fill();
                }
            });

            // 코인 그리기
            coins.forEach(c => {
                if (!c.collected) {
                    ctx.fillStyle = '#f1c40f';
                    ctx.beginPath();
                    ctx.arc(c.x + 10, c.y + 10, 8, 0, Math.PI * 2);
                    ctx.fill();
                    ctx.strokeStyle = '#f39c12';
                    ctx.stroke();
                }
            });

            // 적 그리기 (굼바)
            enemies.forEach(e => {
                if (e.alive) {
                    ctx.fillStyle = '#c0392b';
                    ctx.fillRect(e.x, e.y, e.width, e.height);
                    // 눈 표현
                    ctx.fillStyle = '#ffffff';
                    ctx.fillRect(e.x + 4, e.y + 6, 6, 6);
                    ctx.fillRect(e.x + 20, e.y + 6, 6, 6);
                    ctx.fillStyle = '#000000';
                    ctx.fillRect(e.x + 6, e.y + 8, 3, 3);
                    ctx.fillRect(e.x + 22, e.y + 8, 3, 3);
                }
            });

            // 플레이어 (마리오) 그리기
            ctx.fillStyle = '#e74c3c'; // 빨간 모자/옷
            ctx.fillRect(player.x, player.y, player.width, player.height);
            // 멜빵바지
            ctx.fillStyle = '#2980b9';
            ctx.fillRect(player.x, player.y + 20, player.width, 20);
            // 얼굴
            ctx.fillStyle = '#ffdda1';
            const faceX = player.facing === 'right' ? player.x + 12 : player.x + 2;
            ctx.fillRect(faceX, player.y + 6, 16, 12);

            ctx.restore();

            // UI 그리기는 카메라 영향을 받지 않음 (고정 HUD)
            ctx.fillStyle = '#ffffff';
            ctx.font = 'bold 18px monospace';
            ctx.fillText(`SCORE: ${score}`, 20, 30);
            ctx.fillText(`LIVES: ${'❤️'.repeat(lives)}`, 20, 60);

            // 오버레이 메세지
            if (gameOver) {
                ctx.fillStyle = 'rgba(0, 0, 0, 0.75)';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                ctx.fillStyle = '#e74c3c';
                ctx.font = 'bold 40px sans-serif';
                ctx.textAlign = 'center';
                ctx.fillText('GAME OVER', canvas.width / 2, canvas.height / 2 - 20);
                ctx.fillStyle = '#ffffff';
                ctx.font = '20px sans-serif';
                ctx.fillText('R 키를 눌러 다시 시작하세요', canvas.width / 2, canvas.height / 2 + 30);
                ctx.textAlign = 'left';
            }

            if (gameWin) {
                ctx.fillStyle = 'rgba(0, 0, 0, 0.75)';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                ctx.fillStyle = '#2ecc71';
                ctx.font = 'bold 40px sans-serif';
                ctx.textAlign = 'center';
                ctx.fillText('STAGE CLEAR!', canvas.width / 2, canvas.height / 2 - 20);
                ctx.fillStyle = '#f1c40f';
                ctx.font = '22px sans-serif';
                ctx.fillText(`최종 점수: ${score}점`, canvas.width / 2, canvas.height / 2 + 20);
                ctx.fillStyle = '#ffffff';
                ctx.font = '18px sans-serif';
                ctx.fillText('R 키를 눌러 다시 플레이하세요', canvas.width / 2, canvas.height / 2 + 60);
                ctx.textAlign = 'left';
            }
        }

        function drawBackgroundDetails() {
            // 구름 표현
            ctx.fillStyle = 'rgba(255, 255, 255, 0.7)';
            const clouds = [
                { x: 150, y: 80 }, { x: 500, y: 100 }, { x: 900, y: 60 },
                { x: 1400, y: 90 }, { x: 1900, y: 70 }, { x: 2500, y: 100 }
            ];
            clouds.forEach(c => {
                ctx.beginPath();
                ctx.arc(c.x, c.y, 25, 0, Math.PI * 2);
                ctx.arc(c.x + 20, c.y - 10, 25, 0, Math.PI * 2);
                ctx.arc(c.x + 40, c.y, 25, 0, Math.PI * 2);
                ctx.fill();
            });
        }

        // 게임 루프
        function loop() {
            update();
            draw();
            requestAnimationFrame(loop);
        }

        // 실행 시작
        loop();
    </script>
</body>
</html>
"""

with open("index.html", "w", encoding="utf-8") as f:
    f.write(html_code)

print("index.html created successfully.")
