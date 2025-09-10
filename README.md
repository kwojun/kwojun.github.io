<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>테트리스</title>
    <link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap" rel="stylesheet">
    <style>
        body {
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            background-color: #1a1a1a;
            color: #fff;
            font-family: 'Press Start 2P', cursive;
            flex-direction: column;
            text-align: center;
        }
        .game-container {
            display: grid;
            grid-template-columns: 200px 300px 200px;
            gap: 20px;
            background-color: #333;
            padding: 20px;
            border-radius: 15px;
            box-shadow: 0 0 20px rgba(0, 0, 0, 0.5);
        }
        canvas {
            background-color: #000;
            border: 2px solid #555;
            border-radius: 8px;
        }
        .info-panel {
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            align-items: center;
            padding: 10px;
            color: #ddd;
        }
        .info-box {
            background-color: #222;
            padding: 15px;
            border-radius: 10px;
            border: 1px solid #444;
            width: 100%;
            margin-bottom: 15px;
        }
        .info-box h3 {
            margin-top: 0;
            margin-bottom: 10px;
            font-size: 1em;
            color: #fff;
        }
        .info-box p {
            margin: 5px 0;
            font-size: 0.9em;
        }
        .next-block-preview {
            width: 100px;
            height: 100px;
            background-color: #000;
            border: 2px solid #555;
            border-radius: 5px;
            margin-top: 10px;
            margin-left: auto;
            margin-right: auto;
        }
        .controls-panel {
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            align-items: center;
            padding: 10px;
        }
        .control-button {
            background-color: #444;
            color: #fff;
            border: none;
            padding: 15px;
            font-size: 0.8em;
            margin: 5px;
            cursor: pointer;
            border-radius: 8px;
            transition: background-color 0.2s, transform 0.2s;
            width: 100px;
        }
        .control-button:hover {
            background-color: #666;
            transform: translateY(-2px);
        }
        .control-button:active {
            background-color: #222;
            transform: translateY(0);
        }
        #restartButton {
            background-color: #f44336;
            width: 100%;
            font-size: 1em;
            margin-top: 20px;
        }
        #restartButton:hover {
            background-color: #d32f2f;
        }
        #gameOverScreen {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background-color: rgba(0, 0, 0, 0.8);
            color: #ff0000;
            padding: 40px;
            border-radius: 15px;
            text-shadow: 2px 2px 5px #000;
            font-size: 1.5em;
            display: none;
            border: 2px solid #ff0000;
            box-shadow: 0 0 20px #ff0000;
        }
        @media (max-width: 768px) {
            .game-container {
                grid-template-columns: 1fr;
                gap: 10px;
                padding: 10px;
            }
            .info-panel, .controls-panel {
                flex-direction: row;
                justify-content: center;
                gap: 10px;
            }
            .info-box {
                margin-bottom: 0;
            }
            .next-block-preview {
                display: none;
            }
        }
    </style>
</head>
<body>
    <h1>테트리스</h1>
    <div class="game-container">
        <div class="info-panel">
            <div class="info-box">
                <h3>점수</h3>
                <p id="score">0</p>
            </div>
            <div class="info-box">
                <h3>레벨</h3>
                <p id="level">1</p>
            </div>
            <div class="info-box">
                <h3>다음 블록</h3>
                <canvas id="nextBlockCanvas" class="next-block-preview" width="100" height="100"></canvas>
            </div>
        </div>
        <canvas id="gameCanvas" width="300" height="600"></canvas>
        <div class="controls-panel">
            <div class="info-box">
                <h3>조작법</h3>
                <p>←: 왼쪽 이동</p>
                <p>→: 오른쪽 이동</p>
                <p>↓: 빠르게 내리기</p>
                <p>↑: 회전</p>
                <p>스페이스바: 즉시 내리기</p>
            </div>
            <button class="control-button" id="rotateButton">회전</button>
            <button class="control-button" id="leftButton">←</button>
            <button class="control-button" id="rightButton">→</button>
            <button class="control-button" id="downButton">↓</button>
            <button class="control-button" id="dropButton">즉시 내리기</button>
            <button class="control-button" id="restartButton">다시 시작</button>
        </div>
    </div>
    <div id="gameOverScreen">
        게임 오버!
        <p id="finalScore"></p>
    </div>

    <script>
        document.addEventListener("DOMContentLoaded", () => {
            const gameCanvas = document.getElementById('gameCanvas');
            const nextBlockCanvas = document.getElementById('nextBlockCanvas');
            const gameCtx = gameCanvas.getContext('2d');
            const nextBlockCtx = nextBlockCanvas.getContext('2d');
            const scoreElement = document.getElementById('score');
            const levelElement = document.getElementById('level');
            const gameOverScreen = document.getElementById('gameOverScreen');
            const finalScoreElement = document.getElementById('finalScore');
            const restartButton = document.getElementById('restartButton');
            const rotateButton = document.getElementById('rotateButton');
            const leftButton = document.getElementById('leftButton');
            const rightButton = document.getElementById('rightButton');
            const downButton = document.getElementById('downButton');
            const dropButton = document.getElementById('dropButton');

            const COLS = 10;
            const ROWS = 20;
            const BLOCK_SIZE = 30;

            const SHAPES = [
                // I
                [[[1, 1, 1, 1]], [[0, 1, 0, 0], [0, 1, 0, 0], [0, 1, 0, 0], [0, 1, 0, 0]]],
                // O
                [[[1, 1], [1, 1]]],
                // T
                [[[0, 1, 0], [1, 1, 1]], [[1, 0], [1, 1], [1, 0]], [[1, 1, 1], [0, 1, 0]], [[0, 1], [1, 1], [0, 1]]],
                // L
                [[[0, 0, 1], [1, 1, 1]], [[1, 0], [1, 0], [1, 1]], [[1, 1, 1], [1, 0, 0]], [[1, 1], [0, 1], [0, 1]]],
                // J
                [[[1, 0, 0], [1, 1, 1]], [[1, 1], [1, 0], [1, 0]], [[1, 1, 1], [0, 0, 1]], [[0, 1], [0, 1], [1, 1]]],
                // S
                [[[0, 1, 1], [1, 1, 0]], [[1, 0], [1, 1], [0, 1]]],
                // Z
                [[[1, 1, 0], [0, 1, 1]], [[0, 1], [1, 1], [1, 0]]]
            ];

            const COLORS = ['cyan', 'yellow', 'purple', 'orange', 'blue', 'green', 'red'];

            let board = [];
            let currentBlock = null;
            let nextBlock = null;
            let score = 0;
            let level = 1;
            let fallSpeed = 1000;
            let lastTime = 0;
            let gameOver = false;
            let rotationIndex = 0;

            function createBoard() {
                return Array.from({ length: ROWS }, () => Array(COLS).fill(0));
            }

            function generateNewBlock() {
                const shapeIndex = Math.floor(Math.random() * SHAPES.length);
                const color = COLORS[shapeIndex];
                const shape = SHAPES[shapeIndex];
                
                return {
                    shape: shape,
                    color: color,
                    x: Math.floor(COLS / 2) - Math.floor(shape[0][0].length / 2),
                    y: 0,
                    rotation: 0
                };
            }

            // 블록을 그리는 함수. isGhost가 true이면 투명하게 그립니다.
            function drawBlock(block, ctx, offsetX = 0, offsetY = 0, isGhost = false) {
                if (!block) return;
                const shape = block.shape[block.rotation];
                ctx.fillStyle = block.color;
                if (isGhost) {
                    ctx.globalAlpha = 0.3; // 투명도 설정
                }
                for (let row = 0; row < shape.length; row++) {
                    for (let col = 0; col < shape[row].length; col++) {
                        if (shape[row][col]) {
                            ctx.fillRect((block.x + col + offsetX) * BLOCK_SIZE, (block.y + row + offsetY) * BLOCK_SIZE, BLOCK_SIZE - 1, BLOCK_SIZE - 1);
                        }
                    }
                }
                if (isGhost) {
                    ctx.globalAlpha = 1.0; // 투명도 초기화
                }
            }

            function drawBoard() {
                gameCtx.clearRect(0, 0, gameCanvas.width, gameCanvas.height);
                for (let row = 0; row < ROWS; row++) {
                    for (let col = 0; col < COLS; col++) {
                        if (board[row][col]) {
                            gameCtx.fillStyle = board[row][col];
                            gameCtx.fillRect(col * BLOCK_SIZE, row * BLOCK_SIZE, BLOCK_SIZE - 1, BLOCK_SIZE - 1);
                        }
                    }
                }
            }

            function canMove(block, dx, dy, dRotation = 0) {
                const newShape = block.shape[(block.rotation + dRotation) % block.shape.length];
                for (let row = 0; row < newShape.length; row++) {
                    for (let col = 0; col < newShape[row].length; col++) {
                        if (newShape[row][col]) {
                            const newX = block.x + col + dx;
                            const newY = block.y + row + dy;
                            if (newX < 0 || newX >= COLS || newY >= ROWS || (newY >= 0 && board[newY][newX])) {
                                return false;
                            }
                        }
                    }
                }
                return true;
            }
            
            // 고스트 블록의 y 위치를 찾는 함수
            function findGhostY() {
                let ghostBlock = { ...currentBlock };
                while (canMove(ghostBlock, 0, 1)) {
                    ghostBlock.y++;
                }
                return ghostBlock.y;
            }

            function moveBlock(dx, dy) {
                if (canMove(currentBlock, dx, dy)) {
                    currentBlock.x += dx;
                    currentBlock.y += dy;
                }
            }

            function rotateBlock() {
                if (canMove(currentBlock, 0, 0, 1)) {
                    currentBlock.rotation = (currentBlock.rotation + 1) % currentBlock.shape.length;
                }
            }

            function dropBlock() {
                while (canMove(currentBlock, 0, 1)) {
                    currentBlock.y++;
                    score += 1;
                    scoreElement.innerText = score;
                }
                lockBlock();
            }

            function lockBlock() {
                const shape = currentBlock.shape[currentBlock.rotation];
                for (let row = 0; row < shape.length; row++) {
                    for (let col = 0; col < shape[row].length; col++) {
                        if (shape[row][col]) {
                            if (currentBlock.y + row < 0) {
                                // Game Over
                                gameOver = true;
                                finalScoreElement.innerText = "점수: " + score;
                                gameOverScreen.style.display = 'block';
                                return;
                            }
                            board[currentBlock.y + row][currentBlock.x + col] = currentBlock.color;
                        }
                    }
                }
                checkLines();
                currentBlock = nextBlock;
                nextBlock = generateNewBlock();
                drawNextBlock();
            }

            function checkLines() {
                let linesCleared = 0;
                for (let row = ROWS - 1; row >= 0; row--) {
                    if (board[row].every(cell => cell !== 0)) {
                        board.splice(row, 1);
                        board.unshift(Array(COLS).fill(0));
                        linesCleared++;
                        row++; // Check the same row again since the board shifted down
                    }
                }
                if (linesCleared > 0) {
                    score += linesCleared * 100;
                    scoreElement.innerText = score;
                    if (score >= level * 500) {
                        level++;
                        levelElement.innerText = level;
                        fallSpeed = Math.max(100, fallSpeed - 50); // Increase speed
                    }
                }
            }

            function drawNextBlock() {
                nextBlockCtx.clearRect(0, 0, nextBlockCanvas.width, nextBlockCanvas.height);
                if (nextBlock) {
                    const blockShape = nextBlock.shape[0];
                    const startX = (nextBlockCanvas.width / BLOCK_SIZE - blockShape[0].length) / 2;
                    const startY = (nextBlockCanvas.height / BLOCK_SIZE - blockShape.length) / 2;
                    drawBlock({ ...nextBlock, x: startX, y: startY, rotation: 0 }, nextBlockCtx);
                }
            }
            
            function gameLoop(time = 0) {
                if (gameOver) return;
                const deltaTime = time - lastTime;
                if (deltaTime > fallSpeed) {
                    lastTime = time;
                    if (canMove(currentBlock, 0, 1)) {
                        currentBlock.y++;
                    } else {
                        lockBlock();
                    }
                }
                
                drawBoard();
                
                // 고스트 블록 그리기
                const ghostY = findGhostY();
                drawBlock({ ...currentBlock, y: ghostY }, gameCtx, 0, 0, true);
                
                // 현재 블록 그리기
                drawBlock(currentBlock, gameCtx);

                requestAnimationFrame(gameLoop);
            }

            function handleKeyPress(e) {
                if (gameOver) return;
                switch (e.key) {
                    case 'ArrowLeft':
                        moveBlock(-1, 0);
                        break;
                    case 'ArrowRight':
                        moveBlock(1, 0);
                        break;
                    case 'ArrowDown':
                        moveBlock(0, 1);
                        break;
                    case 'ArrowUp':
                        rotateBlock();
                        break;
                    case ' ':
                        e.preventDefault();
                        dropBlock();
                        break;
                }
                drawBoard();
                drawBlock(currentBlock, gameCtx);
            }

            function handleButtonClick(action) {
                if (gameOver) return;
                switch (action) {
                    case 'rotate':
                        rotateBlock();
                        break;
                    case 'left':
                        moveBlock(-1, 0);
                        break;
                    case 'right':
                        moveBlock(1, 0);
                        break;
                    case 'down':
                        moveBlock(0, 1);
                        break;
                    case 'drop':
                        dropBlock();
                        break;
                }
                drawBoard();
                drawBlock(currentBlock, gameCtx);
            }

            rotateButton.addEventListener('click', () => handleButtonClick('rotate'));
            leftButton.addEventListener('click', () => handleButtonClick('left'));
            rightButton.addEventListener('click', () => handleButtonClick('right'));
            downButton.addEventListener('click', () => handleButtonClick('down'));
            dropButton.addEventListener('click', () => handleButtonClick('drop'));

            function resetGame() {
                gameOverScreen.style.display = 'none';
                board = createBoard();
                score = 0;
                level = 1;
                fallSpeed = 1000;
                scoreElement.innerText = score;
                levelElement.innerText = level;
                gameOver = false;
                lastTime = 0;
                currentBlock = generateNewBlock();
                nextBlock = generateNewBlock();
                drawNextBlock();
                gameLoop();
            }

            restartButton.addEventListener('click', resetGame);
            document.addEventListener('keydown', handleKeyPress);
            
            // Initial setup
            resetGame();
        });
    </script>
</body>
</html>
