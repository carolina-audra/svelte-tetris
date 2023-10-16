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

const cells = new Array<CellStatus>(cellsQty).fill(CellStatus.Empty);
cells[coords(5, 0)] = CellStatus.Active;
cells[coords(5, 1)] = CellStatus.Active;
cells[coords(5, 2)] = CellStatus.Active;
cells[coords(4, 2)] = CellStatus.Active;

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


            if (event.key === "ArrowRight" && coords(column + 1, row) === activeCellIndex) {
                movementBlocked = true;
            } else if (event.key === "ArrowLeft" && coords(column - 1, row) === activeCellIndex) {
                movementBlocked = true;
            }
        }

        if (!movementBlocked) {
            for (const activeCellIndex of activeCellsIndex) {
                cells[activeCellIndex] = CellStatus.Empty;
            }

            for (const activeCellIndex of activeCellsIndex) {
                const [column, row] = coordsFromIndex(activeCellIndex);

            if (event.key === "ArrowRight") {
                cells[coords(column + 1, row)] = CellStatus.Active;

            } else { // ArrowLeft
                cells[coords(column - 1, row)] = CellStatus.Active;
            }
        }
        }
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

<svelte:window
    on:keydown={handleKeyDown}
/>