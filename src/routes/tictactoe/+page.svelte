<svelte:options runes={true} />

<script lang="ts">
	import BackHome from "$lib/components/BackHome.svelte";

    type Player = 'x' | 'o';
    type CellValue = Player | null;

    const winningCombinations: number[][] = [
        [0, 1, 2], [3, 4, 5], [6, 7, 8],
        [0, 3, 6], [1, 4, 7], [2, 5, 8],
        [0, 4, 8], [2, 4, 6]
    ];

    let board = $state<CellValue[]>(Array(9).fill(null));
    let currentPlayer = $state<Player>('x');
    let gameOver = $state(false);
    let winnerMessage = $state('');
    let winningLine = $state<number[]>([]);

    function checkWinner(player: Player) {
        for (const combination of winningCombinations) {
            if (combination.every(index => board[index] === player)) {
                winningLine = combination;
                return true;
            }
        }
        return false;
    }

    function handleCellClick(index: number) {
        if (gameOver || board[index]) return;

        board[index] = currentPlayer;

        if (checkWinner(currentPlayer)) {
            winnerMessage = `Người chơi ${currentPlayer.toUpperCase()} đã chiến thắng!`;
            gameOver = true;
            return;
        }

        if (board.every(cell => cell !== null)) {
            winnerMessage = 'Trận đấu hòa!';
            gameOver = true;
            return;
        }

        currentPlayer = currentPlayer === 'x' ? 'o' : 'x';
    }

    function resetGame() {
        board = Array(9).fill(null);
        currentPlayer = 'x';
        winnerMessage = '';
        gameOver = false;
        winningLine = [];
    }
</script>

<div class="page">
    <BackHome/>
    <div class="glass-header">
        <h1 class="title">TIC<span>TAC</span>TOE</h1>
        <div class="status-bar">
            <div class="player-indicator" class:active={currentPlayer === 'x' && !gameOver}>
                <span class="label">PLAYER X</span>
            </div>
            <div class="vs">VS</div>
            <div class="player-indicator o-side" class:active={currentPlayer === 'o' && !gameOver}>
                <span class="label">PLAYER O</span>
            </div>
        </div>
    </div>

    <div class="game-container">
        {#each board as cell, index (index)}
            <button
                class="cell"
                class:is-winner={winningLine.includes(index)}
                class:x-type={cell === 'x'}
                class:o-type={cell === 'o'}
                data-turn={currentPlayer}
                disabled={gameOver || cell !== null}
                onclick={() => handleCellClick(index)}
            >
                {#if cell}
                    <span class="symbol">{cell.toUpperCase()}</span>
                {/if}
            </button>
        {/each}
    </div>

    <div class="result-area" class:visible={gameOver}>
        <p class="winner-text">{winnerMessage}</p>
        <button class="reset-btn" onclick={resetGame}>
            <span class="icon">↺</span> PLAY AGAIN
        </button>
    </div>
</div>

<style>
    .page {
        height: 100vh;
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        gap: 2rem;
    }

    .glass-header {
        text-align: center;
        background: rgba(255, 255, 255, 0.03);
        padding: 20px 40px;
        border-radius: 20px;
        backdrop-filter: blur(10px);
        border: 1px solid rgba(255, 255, 255, 0.1);
    }

    .title {
        margin: 0 0 1.5rem 0;
        font-size: 2.5rem;
        letter-spacing: 8px;
        font-weight: 900;
        color: #fff;
        text-shadow: 0 0 10px rgba(255, 255, 255, 0.3);
    }

    .title span { color: #00d2ff; }

    .status-bar {
        display: flex;
        align-items: center;
        gap: 20px;
    }

    .player-indicator {
        padding: 8px 20px;
        border-radius: 10px;
        background: rgba(255, 37, 37, 0.1);
        color: #ff2525;
        border: 2px solid transparent;
        transition: all 0.3s ease;
        opacity: 0.4;
    }

    .player-indicator.o-side {
        background: rgba(0, 210, 255, 0.1);
        color: #00d2ff;
    }

    .player-indicator.active {
        opacity: 1;
        border-color: currentColor;
        box-shadow: 0 0 15px currentColor;
        transform: scale(1.1);
    }

    .vs { font-weight: bold; opacity: 0.5; }

    .game-container {
        display: grid;
        grid-template-columns: repeat(3, 1fr);
        gap: 15px;
        perspective: 1000px;
    }

    .cell {
        width: 110px;
        height: 110px;
        background: rgba(255, 255, 255, 0.05);
        border: 1px solid rgba(255, 255, 255, 0.1);
        border-radius: 16px;
        cursor: pointer;
        display: flex;
        justify-content: center;
        align-items: center;
        transition: all 0.2s cubic-bezier(0.175, 0.885, 0.32, 1.275);
        position: relative;
    }

    .cell:not(:disabled):hover::after {
        content: attr(data-turn);
        text-transform: uppercase;
        opacity: 0.15;
        font-size: 3rem;
        font-weight: bold;
        color: white;
    }

    .cell:hover:not(:disabled) {
        background: rgba(255, 255, 255, 0.1);
        transform: translateY(-5px) scale(1.02);
    }

    .cell:active { transform: scale(0.9); }

    .symbol {
        font-size: 3.5rem;
        font-weight: 900;
        animation: appear 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275) forwards;
    }

    @keyframes appear {
        from { opacity: 0; transform: scale(0) rotate(-45deg); }
        to { opacity: 1; transform: scale(1) rotate(0); }
    }

    .x-type .symbol { color: #ff2525; text-shadow: 0 0 20px rgba(255, 37, 37, 0.5); }
    .o-type .symbol { color: #00d2ff; text-shadow: 0 0 20px rgba(0, 210, 255, 0.5); }

    .cell.is-winner {
        background: rgba(255, 255, 255, 0.2);
        animation: pulse 1s infinite alternate;
        border: 2px solid white;
    }

    @keyframes pulse {
        from { box-shadow: 0 0 10px rgba(255,255,255,0.2); }
        to { box-shadow: 0 0 30px rgba(255,255,255,0.6); }
    }

    .result-area {
        display: flex;
        flex-direction: column;
        align-items: center;
        opacity: 0;
        transform: translateY(20px);
        transition: all 0.5s ease;
        pointer-events: none;
    }

    .result-area.visible {
        opacity: 1;
        transform: translateY(0);
        pointer-events: all;
    }

    .winner-text {
        font-size: 20px;
        font-weight: bold;
        margin-bottom: 1.5rem;
        text-transform: uppercase;
        letter-spacing: 2px;
        background: linear-gradient(to right, #00d2ff, #3a7bd5);
        -webkit-background-clip: text;
        -webkit-text-fill-color: transparent;
    }

    .reset-btn {
        background: #fff;
        color: #0f0c29;
        border: none;
        padding: 15px 40px;
        border-radius: 50px;
        font-weight: 900;
        font-size: 1rem;
        cursor: pointer;
        display: flex;
        align-items: center;
        gap: 10px;
        transition: all 0.3s ease;
        box-shadow: 0 10px 20px rgba(0,0,0,0.3);
    }

    .reset-btn:hover {
        transform: translateY(-3px) scale(1.05);
        box-shadow: 0 15px 30px rgba(0,0,0,0.4);
    }

    .reset-btn .icon { font-size: 1.4rem; }

    @media (max-width: 500px) {
        .cell { width: 90px; height: 90px; }
        .title { font-size: 1.8rem; }
    }
</style>