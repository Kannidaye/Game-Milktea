<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>合成奶茶</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f4f4f4;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
        }

        #game-container {
            text-align: center;
        }

        #board {
            display: grid;
            grid-template-columns: repeat(4, 80px);
            grid-template-rows: repeat(4, 80px);
            gap: 10px;
            background-color: #bbada0;
            padding: 10px;
            border-radius: 6px;
        }

        .cell {
            background-color: rgba(238, 228, 218, 0.35);
            border-radius: 3px;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 20px;
            font-weight: bold;
        }

        #score {
            margin-top: 20px;
            font-size: 24px;
        }

        #status {
            margin-top: 10px;
            font-size: 20px;
            color: red;
        }

        #restart {
            margin-top: 20px;
            padding: 10px 20px;
            font-size: 18px;
            background-color: #8f7a66;
            color: white;
            border: none;
            border-radius: 3px;
            cursor: pointer;
        }
    </style>
</head>

<body>
    <div id="game-container">
        <div id="board"></div>
        <div id="score">合成次数: 0</div>
        <div id="status"></div>
        <button id="restart">重新开始</button>
    </div>
    <script>
        // 优化后的奶茶原料数组
        const ingredients = [
            "珍珠", "椰果", "红茶", "牛奶",
            "双份珍珠", "双份椰果", "茶奶基底",
            "珍珠奶茶", "椰果奶茶", "奶茶"
        ];

        const boardSize = 4;
        let board = Array.from({ length: boardSize }, () => Array(boardSize).fill(null));
        let score = 0;
        let gameOver = false;
        let gameWon = false;

        const boardElement = document.getElementById('board');
        const scoreElement = document.getElementById('score');
        const statusElement = document.getElementById('status');
        const restartButton = document.getElementById('restart');

        function initGame() {
            board = Array.from({ length: boardSize }, () => Array(boardSize).fill(null));
            score = 0;
            gameOver = false;
            gameWon = false;
            statusElement.textContent = '';
            scoreElement.textContent = `合成次数: ${score}`;
            addRandomIngredient();
            addRandomIngredient();
            renderBoard();
        }

        function addRandomIngredient() {
            const emptyCells = [];
            for (let i = 0; i < boardSize; i++) {
                for (let j = 0; j < boardSize; j++) {
                    if (board[i][j] === null) {
                        emptyCells.push({ i, j });
                    }
                }
            }
            if (emptyCells.length > 0) {
                const randomIndex = Math.floor(Math.random() * emptyCells.length);
                const { i, j } = emptyCells[randomIndex];
                board[i][j] = ingredients[Math.floor(Math.random() * 4)];
            }
        }

        function renderBoard() {
            boardElement.innerHTML = '';
            for (let i = 0; i < boardSize; i++) {
                for (let j = 0; j < boardSize; j++) {
                    const cell = document.createElement('div');
                    cell.classList.add('cell');
                    if (board[i][j]) {
                        cell.textContent = board[i][j];
                    }
                    boardElement.appendChild(cell);
                }
            }
        }

        function move(direction) {
            if (gameOver || gameWon) return;
            let moved = false;
            let newBoard;
            switch (direction) {
                case 'up':
                    newBoard = moveUp();
                    break;
                case 'down':
                    newBoard = moveDown();
                    break;
                case 'left':
                    newBoard = moveLeft();
                    break;
                case 'right':
                    newBoard = moveRight();
                    break;
            }
            if (JSON.stringify(newBoard) !== JSON.stringify(board)) {
                board = newBoard;
                moved = true;
                addRandomIngredient();
                renderBoard();
                checkGameStatus();
            }
            return moved;
        }

        function moveUp() {
            let newBoard = Array.from({ length: boardSize }, () => Array(boardSize).fill(null));
            for (let j = 0; j < boardSize; j++) {
                let newColumn = [];
                for (let i = 0; i < boardSize; i++) {
                    if (board[i][j]) {
                        newColumn.push(board[i][j]);
                    }
                }
                newColumn = merge(newColumn);
                for (let i = 0; i < newColumn.length; i++) {
                    newBoard[i][j] = newColumn[i];
                }
            }
            return newBoard;
        }

        function moveDown() {
            let newBoard = Array.from({ length: boardSize }, () => Array(boardSize).fill(null));
            for (let j = 0; j < boardSize; j++) {
                let newColumn = [];
                for (let i = boardSize - 1; i >= 0; i--) {
                    if (board[i][j]) {
                        newColumn.push(board[i][j]);
                    }
                }
                newColumn = merge(newColumn);
                newColumn.reverse();
                for (let i = boardSize - newColumn.length; i < boardSize; i++) {
                    newBoard[i][j] = newColumn[i - (boardSize - newColumn.length)];
                }
            }
            return newBoard;
        }

        function moveLeft() {
            let newBoard = Array.from({ length: boardSize }, () => Array(boardSize).fill(null));
            for (let i = 0; i < boardSize; i++) {
                let newRow = [];
                for (let j = 0; j < boardSize; j++) {
                    if (board[i][j]) {
                        newRow.push(board[i][j]);
                    }
                }
                newRow = merge(newRow);
                for (let j = 0; j < newRow.length; j++) {
                    newBoard[i][j] = newRow[j];
                }
            }
            return newBoard;
        }

        function moveRight() {
            let newBoard = Array.from({ length: boardSize }, () => Array(boardSize).fill(null));
            for (let i = 0; i < boardSize; i++) {
                let newRow = [];
                for (let j = boardSize - 1; j >= 0; j--) {
                    if (board[i][j]) {
                        newRow.push(board[i][j]);
                    }
                }
                newRow = merge(newRow);
                newRow.reverse();
                for (let j = boardSize - newRow.length; j < boardSize; j++) {
                    newBoard[i][j] = newRow[j - (boardSize - newRow.length)];
                }
            }
            return newBoard;
        }

        function merge(arr) {
            let newArr = [];
            let i = 0;
            while (i < arr.length) {
                if (i + 1 < arr.length && arr[i] === arr[i + 1]) {
                    const index = ingredients.indexOf(arr[i]);
                    if (index < ingredients.length - 1) {
                        newArr.push(ingredients[index + 1]);
                        score++;
                        scoreElement.textContent = `合成次数: ${score}`;
                    }
                    i += 2;
                } else {
                    newArr.push(arr[i]);
                    i++;
                }
            }
            return newArr;
        }

        function checkGameStatus() {
            if (score === 8) {
                gameWon = true;
                statusElement.textContent = '游戏胜利！';
            } else {
                let hasEmptyCell = false;
                let canMerge = false;
                for (let i = 0; i < boardSize; i++) {
                    for (let j = 0; j < boardSize; j++) {
                        if (board[i][j] === null) {
                            hasEmptyCell = true;
                        }
                        if (i > 0 && board[i][j] === board[i - 1][j]) {
                            canMerge = true;
                        }
                        if (j > 0 && board[i][j] === board[i][j - 1]) {
                            canMerge = true;
                        }
                    }
                }
                if (!hasEmptyCell && !canMerge) {
                    gameOver = true;
                    statusElement.textContent = '游戏失败！';
                }
            }
        }

        document.addEventListener('keydown', function (event) {
            switch (event.key) {
                case 'ArrowUp':
                    move('up');
                    break;
                case 'ArrowDown':
                    move('down');
                    break;
                case 'ArrowLeft':
                    move('left');
                    break;
                case 'ArrowRight':
                    move('right');
                    break;
            }
        });

        let touchStartX = 0;
        let touchStartY = 0;
        document.addEventListener('touchstart', function (event) {
            touchStartX = event.touches[0].clientX;
            touchStartY = event.touches[0].clientY;
        });
        document.addEventListener('touchend', function (event) {
            const touchEndX = event.changedTouches[0].clientX;
            const touchEndY = event.changedTouches[0].clientY;
            const dx = touchEndX - touchStartX;
            const dy = touchEndY - touchStartY;
            if (Math.abs(dx) > Math.abs(dy)) {
                if (dx > 0) {
                    move('right');
                } else {
                    move('left');
                }
            } else {
                if (dy > 0) {
                    move('down');
                } else {
                    move('up');
                }
            }
        });

        restartButton.addEventListener('click', initGame);

        initGame();
    </script>
</body>

</html>
