<script>
	let rows = 10;
	let cols = 10;
	let mines = 15;
	let gameOver = false;

	// Initialize board logic
	let board = Array(rows)
		.fill(null)
		.map(() =>
			Array(cols).fill({
				revealed: false,
				mine: false,
				count: 0,
				flagged: false,
			})
		);

	function initBoard() {
		board = board.map((row) =>
			row.map(() => ({
				revealed: false,
				mine: false,
				count: 0,
				flagged: false,
			}))
		);

		// Place mines
		let placedMines = 0;
		while (placedMines < mines) {
			const r = Math.floor(Math.random() * rows);
			const c = Math.floor(Math.random() * cols);
			if (!board[r][c].mine) {
				board[r][c].mine = true;
				placedMines++;
			}
		}

		// Calculate counts
		for (let r = 0; r < rows; r++) {
			for (let c = 0; c < cols; c++) {
				if (!board[r][c].mine) {
					let count = 0;
					for (let dr of [-1, 0, 1]) {
						for (let dc of [-1, 0, 1]) {
							const nr = r + dr;
							const nc = c + dc;
							if (
								nr >= 0 &&
								nr < rows &&
								nc >= 0 &&
								nc < cols &&
								board[nr][nc].mine
							) {
								count++;
							}
						}
					}
					board[r][c].count = count;
				}
			}
		}
	}

	initBoard();

	function reveal(row, col) {
		if (gameOver || board[row][col].revealed || board[row][col].flagged)
			return;
		board[row][col].revealed = true;

		if (board[row][col].mine) {
			// Show all mines on game over
			board.forEach((row) =>
				row.forEach((cell) => {
					if (cell.mine) cell.revealed = true;
				})
			);
			gameOver = true;
			return;
		}

		// Automatically reveal neighbors if count is 0
		if (board[row][col].count === 0) {
			for (let dr of [-1, 0, 1]) {
				for (let dc of [-1, 0, 1]) {
					const nr = row + dr;
					const nc = col + dc;
					if (nr >= 0 && nr < rows && nc >= 0 && nc < cols) {
						reveal(nr, nc);
					}
				}
			}
		}
	}

	function toggleFlag(row, col) {
		if (!gameOver && !board[row][col].revealed) {
			board[row][col].flagged = !board[row][col].flagged;
		}
	}

	function resetGame() {
		gameOver = false;
		initBoard();
	}
</script>

<h1>Minesweeper</h1>
<button on:click={resetGame}>Reset</button>
<table>
	<tbody>
		{#each board as row, r}
			<tr>
				{#each row as cell, c}
					<td
						class="{cell.revealed ? 'revealed' : ''} {cell.mine &&
						cell.revealed
							? 'mine'
							: ''} {cell.flagged ? 'flagged' : ''}"
						on:click={() => reveal(r, c)}
						on:contextmenu|preventDefault={() => toggleFlag(r, c)}
					>
						{#if cell.revealed}
							{cell.mine
								? "💣"
								: cell.count > 0
									? cell.count
									: ""}
						{:else if cell.flagged}
							🚩
						{/if}
					</td>
				{/each}
			</tr>
		{/each}
	</tbody>
</table>

<style>
	table {
		border-collapse: collapse;
		margin: auto;
	}
	td {
		width: 30px;
		height: 30px;
		text-align: center;
		border: 1px solid #ccc;
		cursor: pointer;
		background-color: lightgray;
	}
	td.revealed {
		background-color: white;
		cursor: default;
	}
	td.mine {
		background-color: red;
		color: white;
	}
	td.flagged {
		background-color: yellow;
	}
</style>
