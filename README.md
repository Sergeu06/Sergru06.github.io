<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tetris</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            background-color: #000;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            overflow: hidden;
            color: #fff;
        }
        .game-container {
            position: relative;
            display: flex;
            flex-direction: row;
            width: 95vw;
            height: 95vh;
            max-width: 600px;
            max-height: 800px;
            border: 1px solid #fff;
        }
        .game-board {
            position: relative;
            flex: 0 0 75%;
            border-right: 1px solid #fff;
            display: grid;
            grid-template-columns: repeat(10, 1fr);
            grid-template-rows: repeat(20, 1fr);
        }
        .cell {
            border: 1px solid #333;
            background-color: #222;
        }
        .info {
            position: relative;
            flex: 1;
            padding: 10px;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: space-around;
            text-align: center;
        }
        #next-block {
            border: 1px solid #fff;
            background-color: #000;
            max-width: 100px;
            max-height: 100px;
            width: 100%;
            height: auto;
        }
        canvas {
            border: none;
            background-color: #000;
            width: 100%;
            height: auto;
            max-height: 50%;
            display: block;
        }

        @media screen and (max-width: 600px) {
            h2, p {
                font-size: 3vw;
            }
        }

        .start-button {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            padding: 10px 20px;
            background-color: #fff;
            color: #000;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-size: 1.5rem;
        }
    </style>
</head>
<body>
    <div class="game-container">
        <div class="game-board" id="game-board">
        </div>
        <div class="info">
            <h2>Счет:</h2>
            <p id="score">0</p>
            <h2>Следующая фигура:</h2>
            <canvas id="next-block" width="100" height="100"></canvas>
        </div>
    </div>
    <button class="start-button" onclick="startGame()">Начать игру</button>

<script>
    const rows = 20;
    const cols = 10;
    let currentShape;
    let interval;
    let gameBoard = [];
    let currentX;
    let currentY;
    let dropInterval = 1000; // Интервал падения фигуры по умолчанию
    let dropIntervalId;

    // Функция для создания случайной фигуры
    function createRandomShape() {
        const shapes = [
            [
                [1, 1, 1, 1] // I-фигура
            ],
            [
                [1, 1, 1],
                [0, 1, 0] // T-фигура
            ],
            [
                [1, 1, 1],
                [1, 0, 0] // L-фигура
            ],
            [
                [1, 1, 1],
                [0, 0, 1] // J-фигура
            ],
            [
                [1, 1],
                [1, 1] // O-фигура
            ],
            [
                [0, 1, 1],
                [1, 1, 0] // Z-фигура
            ],
            [
                [1, 1, 0],
                [0, 1, 1] // S-фигура
            ]
        ];

        // Случайным образом выбираем одну из фигур
        const randomIndex = Math.floor(Math.random() * shapes.length);
        return shapes[randomIndex];
    }

    // Функция для начала игры
    function startGame() {
        currentShape = createRandomShape();
        clearInterval(interval);
        gameBoard = [];
        for (let i = 0; i < rows; i++) {
            gameBoard[i] = [];
            for (let j = 0; j < cols; j++) {
                gameBoard[i][j] = null;
            }
        }
        interval = setInterval(moveDown, dropInterval);
        currentX = Math.floor(cols / 2) - Math.floor(currentShape[0].length / 2);
        currentY = 0;
        drawShape(currentShape, currentX, currentY);
    }

    // Функция для движения фигуры вниз
    function moveDown() {
        if (!checkCollision(currentShape, currentX, currentY + 1)) {
            clearShape(currentShape, currentX, currentY);
            currentY++;
            drawShape(currentShape, currentX, currentY);
        } else {
            mergeShape(currentShape, currentX, currentY);
            clearInterval(interval); // Останавливаем интервал
            startGame(); // Запускаем новую фигуру
        }
    }

    // Функция для ускоренного падения фигуры
    function moveDownFaster() {
        clearInterval(interval); // Останавливаем текущий интервал
        dropIntervalId = setInterval(moveDown, 50); // Устанавливаем новый интервал для быстрого падения
    }

    // Функция для возврата обычного интервала падения фигуры
    function resetDropInterval() {
        clearInterval(dropIntervalId); // Останавливаем быстрое падение
        interval = setInterval(moveDown, dropInterval); // Возвращаем обычный интервал падения
    }

    // Функция для проверки столкновения фигуры с другими фигурами или краем поля
    function checkCollision(shape, x, y) {
        for (let i = 0; i < shape.length; i++) {
            for (let j = 0; j < shape[i].length; j++) {
                if (shape[i][j]) {
                    // Проверка столкновения с краем поля
                    if (y + i >= rows || x + j < 0 || x + j >= cols) {
                        return true;
                    }
                    // Проверка столкновения с другой фигурой
                    if (gameBoard[y + i][x + j] !== null) {
                        return true;
                    }
                }
            }
        }
        return false;
    }

    // Функция для отрисовки фигуры на игровом поле
    function drawShape(shape, x, y) {
        for (let i = 0; i < shape.length; i++) {
            for (let j = 0; j < shape[i].length; j++) {
                if (shape[i][j]) {
                    const cell = document.createElement('div');
                    cell.classList.add('cell');
                    cell.style.gridColumn = x + j + 1;
                    cell.style.gridRow = y + i + 1;
                    document.getElementById('game-board').appendChild(cell);
                }
            }
        }
    }

    // Функция для очистки фигуры с игрового поля
    function clearShape(shape, x, y) {
        const cells = document.querySelectorAll('.cell');
        cells.forEach(cell => {
            if (parseInt(cell.style.gridRow) > y && parseInt(cell.style.gridColumn) > x &&
                parseInt(cell.style.gridColumn) <= x + shape[0].length &&
                parseInt(cell.style.gridRow) <= y + shape.length) {
                cell.remove();
            }
        });
    }

    // Функция для объединения фигуры с игровым полем
    function mergeShape(shape, x, y) {
        for (let i = 0; i < shape.length; i++) {
            for (let j = 0; j < shape[i].length; j++) {
                if (shape[i][j]) {
                    const cell = document.createElement('div');
                    cell.classList.add('cell');
                    cell.style.gridColumn = x + j + 1;
                    cell.style.gridRow = y + i + 1;
                    document.getElementById('game-board').appendChild(cell);
                    gameBoard[y + i][x + j] = cell;
                }
            }
        }
    }

    // Начать игру при загрузке страницы
    startGame();

    // Обработчики событий для управления фигурой
    document.addEventListener('keydown', event => {
        if (event.key === 'ArrowLeft') {
            // Движение влево
        } else if (event.key === 'ArrowRight') {
            // Движение вправо
        } else if (event.key === 'ArrowDown') {
            moveDownFaster(); // Ускоренное падение
        } else if (event.key === 'ArrowUp') {
            // Поворот по часовой стрелке
        }
    });

    document.addEventListener('keyup', event => {
        if (event.key === 'ArrowDown') {
            resetDropInterval(); // Возврат обычного интервала падения
        }
    });
</script>

</body>
</html>
