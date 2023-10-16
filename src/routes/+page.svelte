<script lang="ts">
const colsQty = 10;
const rowsQty = 17;
const cellsQty = colsQty * rowsQty;

enum CellStatus {
    Empty = 0,
    Active,
    Fixed
};

function coords(x: number, y: number): number {
    x = Math.max(0, x);
    x = Math.min(x, colsQty - 1);

    y = Math.max(0, y);
    y = Math.min(y, rowsQty - 1);

    return (y * colsQty) + x;
};
function coordsFromIndex(index: number) {
    const row = Math.floor(index / colsQty);
    const column = index - (row * colsQty);

    return [column, row];
}

function getCell(x: number, y: number) {
    return cells[coords(x, y)];
}
function setCell(x: number, y: number, value: CellStatus) {
    cells[coords(x, y)] = value;
}
const cells = new Array<CellStatus>(cellsQty).fill(CellStatus.Empty);


const pieces = [
    [ // I piece
        [
            [0, 0, 0, 0],
            [0, 0, 0, 0],
            [1, 1, 1, 1],
            [0, 0, 0, 0]
        ],
        [
            [0, 0, 1, 0],
            [0, 0, 1, 0],
            [0, 0, 1, 0],
            [0, 0, 1, 0]
        ]
    ],
    [ // O piece
        [
            [0, 0, 0, 0],
            [0, 1, 1, 0],
            [0, 1, 1, 0],
            [0, 0, 0, 0]
        ]
    ],
    [ // J piece
        [
            [0, 0, 0],
            [1, 1, 1],
            [0, 0, 1]
        ],
        [
            [0, 1, 0],
            [0, 1, 0],
            [1, 1, 0]
        ],
        [
            [1, 0, 0],
            [1, 1, 1],
            [0, 0, 0]
        ],
        [
            [0, 1, 1],
            [0, 1, 0],
            [0, 1, 0]
        ]
    ],
    [ // L piece
        [
            [0, 0, 0],
            [1, 1, 1],
            [1, 0, 0]
        ],
        [
            [1, 1, 0],
            [0, 1, 0],
            [0, 1, 0]
        ],
        [
            [0, 0, 1],
            [1, 1, 1],
            [0, 0, 0]
        ],
        [
            [0, 1, 0],
            [0, 1, 0],
            [0, 1, 1]
        ]
    ],
    [ // S piece
        [
            [0, 0, 0],
            [0, 1, 1],
            [1, 1, 0]
        ],
        [
            [0, 1, 0],
            [0, 1, 1],
            [0, 0, 1]
        ],
        [
            [0, 0, 0],
            [0, 1, 1],
            [1, 1, 0]
        ],
        [
            [0, 1, 0],
            [0, 1, 1],
            [0, 0, 1]
        ]
    ],
    [ // T piece
        [
            [0, 0, 0],
            [1, 1, 1],
            [0, 1, 0]
        ],
        [
            [0, 1, 0],
            [1, 1, 0],
            [0, 1, 0]
        ],
        [
            [0, 1, 0],
            [1, 1, 1],
            [0, 0, 0]
        ],
        [
            [0, 1, 0],
            [0, 1, 1],
            [0, 1, 0]
        ]
    ],
    [ // Z piece
        [
            [0, 0, 0],
            [1, 1, 0],
            [0, 1, 1]
        ],
        [
            [0, 0, 1],
            [0, 1, 1],
            [0, 1, 0]
        ],
        [
            [0, 0, 0],
            [1, 1, 0],
            [0, 1, 1]
        ],
        [
            [0, 0, 1],
            [0, 1, 1],
            [0, 1, 0]
        ]
    ]
];


let latestPieceIndex = 0;
let latestPieceRotation = 0;
let latestPieceX = 0;
let latestPieceY = 0;

function initWithRandomPiece() {
    cells.fill(CellStatus.Empty);
    latestPieceIndex = Math.floor(Math.random() * pieces.length);
    latestPieceRotation = Math.floor(Math.random() * pieces[latestPieceIndex].length);
    const pieceByPos = pieces[latestPieceIndex][latestPieceRotation];
    
    const centerX = Math.floor(colsQty / 2 - (pieceByPos[0].length / 2));
    latestPieceX = centerX;

    for (let y = 0; y < pieceByPos.length; y++) {
        for (let x = 0; x < pieceByPos[y].length; x++) {

            if (pieceByPos[y][x]) setCell(centerX + x, y, CellStatus.Active);
        }
    }
}
initWithRandomPiece();

function handleKeyDown(event: KeyboardEvent) {

    if (["ArrowLeft", "ArrowRight"].includes(event.key)) {

        const activeCellsIndex: number[] = [];

        for (let cellIndex = 0; cellIndex < cells.length; cellIndex++) {
            if (cells[cellIndex] === CellStatus.Active) {
                activeCellsIndex.push(cellIndex);
            }
        }

        let movementBlocked = false;

        for (const activeCellIndex of activeCellsIndex) {
            const [column, row] = coordsFromIndex(activeCellIndex);


            let desiredColumn = column;
            if (event.key === "ArrowRight") {
                desiredColumn = column + 1;
            } else { // ArrowLeft
                desiredColumn = column - 1;
            }

            if (
                coords(desiredColumn, row) === activeCellIndex
                || getCell(desiredColumn, row) === CellStatus.Fixed
            ) {
                movementBlocked = true;
            }
        }

        if (!movementBlocked) {
            for (const activeCellIndex of activeCellsIndex) {
                cells[activeCellIndex] = CellStatus.Empty;
            }
            if (event.key === "ArrowRight") {
                latestPieceX++;
            } else { // ArrowLeft
                latestPieceX--;
            }

            for (const activeCellIndex of activeCellsIndex) {
                const [column, row] = coordsFromIndex(activeCellIndex);

                if (event.key === "ArrowRight") {
                setCell(column + 1, row, CellStatus.Active);

            } else { // ArrowLeft
                setCell(column - 1, row, CellStatus.Active);
            }
        }
        }
    } else if (event.key === " ") {

    }
}
</script>

<style>
.grid {
    display: grid;
    align-items: center;
    justify-content: center;
    grid-template-columns: repeat(var(--cols), min-content);
    grid-template-rows: repeat(var(--rows), min-content);
    gap: 1px;
    background-color: #888;
}
.cell {
    background-color: #444;
    border-bottom: none;
    border-right: none;
    width: 24px;
    height: 24px;
}
.cell.active {
    background-color: hsl(34, 59%, 54%);
}
</style>
<div style="--cols: {colsQty}; --rows: {rowsQty}">
    <div class="grid">
        {#each cells as cell}
            <div class={"cell " + (cell !== CellStatus.Empty ? "active" : "")}></div>
        {/each}
    </div>
</div>

<button on:click={initWithRandomPiece} type="button">Reset</button><br>
Piece index: {latestPieceIndex}<br>
Piece X: { latestPieceX }<br> 
Rotation: { latestPieceIndex }<br>

<svelte:window
    on:keydown={handleKeyDown}
/>