<script lang="ts">
    import { onMount } from 'svelte';
    import merryChristmasTrack from '$lib/assets/merry-christmas.mp3';
    import BackHome from '$lib/components/BackHome.svelte';

    // --- Giữ nguyên Logic ---
    const TREE_HEIGHT = 15;
    const SNOWFLAKE_INTERVAL_MS  = 200; // Tăng nhẹ để mượt hơn
    const TREE_REFRESH_INTERVAL_MS = 600;
    const currentYear = new Date().getFullYear();
    const yearDigits = String(currentYear).split('');

    interface TreeCell { digit: '0' | '1'; color: string; glow: boolean; }
    interface Snowflake { id: number; left: string; duration: string; size: string; opacity: number; }

    let treeRows = $state<TreeCell[][]>([]);
    let snowflakes = $state<Snowflake[]>([]);
    let isPlaying = $state(false);
    let audioNotice = $state('Click to play festive music');

    let snowId = 0;
    let audioEl: HTMLAudioElement | null = null;

    function generateTree(): void {
        const rows: TreeCell[][] = [];
        for (let i = 0; i < TREE_HEIGHT; i++) {
            const row: TreeCell[] = [];
            for (let j = 0; j < 2 * i + 1; j++) {
                const rand = Math.random();
                const isOrnament = rand < 0.12;
                row.push({
                    digit: Math.random() < 0.5 ? '0' : '1',
                    color: isOrnament ? (rand < 0.06 ? '#ff3e00' : '#ffcc00') : '#10b981',
                    glow: isOrnament
                });
            }
            rows.push(row);
        }
        treeRows = rows;
    }

    function spawnSnowflake(): void {
        const id = ++snowId;
        const flake: Snowflake = {
            id,
            left: `${Math.random() * 100}%`,
            duration: `${Math.random() * 4 + 6}s`,
            size: `${Math.random() * 6 + 4}px`,
            opacity: Math.random() * 0.5 + 0.3
        };
        snowflakes = [...snowflakes, flake];
        setTimeout(() => { snowflakes = snowflakes.filter(s => s.id !== id); }, 10000);
    }

    async function toggleAudio() {
        if (!audioEl) return;
        if (isPlaying) { audioEl.pause(); isPlaying = false; } 
        else { await audioEl.play(); isPlaying = true; }
    }

    onMount(() => {
        generateTree();
        const treeTimer = setInterval(generateTree, TREE_REFRESH_INTERVAL_MS);
        const snowTimer = setInterval(spawnSnowflake, SNOWFLAKE_INTERVAL_MS);
        return () => { clearInterval(treeTimer); clearInterval(snowTimer); };
    });
</script>

<div class="christmas-page">
    <BackHome />
    
    <!-- Snow layer -->
    <div class="snow-container" aria-hidden="true">
        {#each snowflakes as flake (flake.id)}
            <div class="snow" style="left:{flake.left}; animation-duration:{flake.duration}; width:{flake.size}; height:{flake.size}; opacity:{flake.opacity}"></div>
        {/each}
    </div>

    <!-- UI Controls -->
    <div class="controls">
        <button class="music-toggle" class:active={isPlaying} onclick={toggleAudio}>
            <div class="visualizer" class:playing={isPlaying}>
                <span></span><span></span><span></span>
            </div>
            {isPlaying ? 'PAUSE MUSIC' : 'PLAY MUSIC'}
        </button>
        <p class="notice">{isPlaying ? 'Merry Christmas! 🎄' : audioNotice}</p>
    </div>

    <audio bind:this={audioEl} loop src={merryChristmasTrack}></audio>

    <main class="scene">
        <!-- Digital Tree -->
        <div class="tree-wrapper">
            <div class="star-glow">★</div>
            <div class="binary-tree">
                {#each treeRows as row, i (i)}
                    <div class="row">
                        {#each row as cell, j (j)}
                            <span class="cell" class:glow={cell.glow} style="color: {cell.color}">{cell.digit}</span>
                        {/each}
                    </div>
                {/each}
            </div>
            <div class="trunk"></div>
        </div>

        <!-- Sleek Card -->
        <div class="info-card">
            <h2 class="title">Digital Christmas</h2>
            <div class="year-grid">
                {#each yearDigits as d, i (i)}
                    <span class="digit">{d}</span>
                {/each}
            </div>
            <div class="author">
                <span class="label">System Admin:</span>
                <span class="val">Nguyen Quy Coder</span>
            </div>
        </div>
    </main>
</div>

<style>
    /* Chống rò rỉ: Chỉ áp dụng trong class này */
    .christmas-page {
        position: relative;
        width: 100%;
        min-height: 100vh;
        background: radial-gradient(circle at 50% 40%, #064e3b 0%, #020617 100%);
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        overflow: hidden;
        color: #fff;
        font-family: 'Inter', system-ui, sans-serif;
    }

    /* Tuyệt đối không treo: Snow nhẹ nhàng dùng GPU */
    .snow-container {
        position: absolute;
        inset: 0;
        pointer-events: none;
    }
    .snow {
        position: absolute;
        top: -10px;
        background: #fff;
        border-radius: 50%;
        filter: blur(1px);
        animation: fall linear forwards;
    }
    @keyframes fall {
        to { transform: translateY(105vh); }
    }

    /* Scene Layout */
    .scene {
        display: flex;
        flex-direction: column;
        align-items: center;
        gap: 3rem;
        z-index: 10;
    }

    /* Binary Tree Styling */
    .tree-wrapper {
        display: flex;
        flex-direction: column;
        align-items: center;
        filter: drop-shadow(0 0 20px rgba(16, 185, 129, 0.2));
    }

    .star-glow {
        font-size: 3rem;
        color: #fbbf24;
        text-shadow: 0 0 20px #fbbf24;
        animation: pulse 2s infinite;
        margin-bottom: -0.5rem;
    }

    .binary-tree {
        font-family: 'JetBrains Mono', monospace;
        font-size: 1.1rem;
        line-height: 1.1;
        text-align: center;
        letter-spacing: 0.1em;
    }

    .cell {
        transition: color 0.4s ease;
        display: inline-block;
        padding: 0 1px;
    }

    .cell.glow {
        text-shadow: 0 0 8px currentColor;
        font-weight: 900;
    }

    .trunk {
        width: 40px;
        height: 60px;
        background: linear-gradient(to bottom, #1e293b, #020617);
        border-radius: 4px;
        margin-top: 5px;
    }

    .info-card {
        background: rgba(15, 23, 42, 0.6);
        border: 1px solid rgba(255, 255, 255, 0.1);
        padding: 1.5rem 2.5rem;
        border-radius: 1rem;
        backdrop-filter: blur(8px);
        text-align: center;
    }

    .title {
        font-size: 0.8rem;
        text-transform: uppercase;
        letter-spacing: 3px;
        color: #94a3b8;
        margin: 0 0 1rem;
    }

    .year-grid {
        display: flex;
        gap: 0.5rem;
        justify-content: center;
        margin-bottom: 1.5rem;
    }

    .digit {
        font-size: 1.5rem;
        font-weight: 800;
        background: rgba(255, 255, 255, 0.05);
        border: 1px solid rgba(255, 255, 255, 0.1);
        padding: 0.5rem 0.8rem;
        border-radius: 0.5rem;
        color: #10b981;
    }

    .author {
        font-size: 0.8rem;
        border-top: 1px solid rgba(255, 255, 255, 0.05);
        padding-top: 1rem;
    }
    .author .label { color: #475569; margin-right: 5px; }
    .author .val { color: #f8fafc; font-weight: 600; }

    .controls {
        position: fixed;
        bottom: 2rem;
        right: 2rem;
        display: flex;
        flex-direction: column;
        align-items: flex-end;
        z-index: 100;
    }

    .music-toggle {
        display: flex;
        align-items: center;
        gap: 12px;
        background: #fff;
        color: #000;
        border: none;
        padding: 0.8rem 1.2rem;
        border-radius: 2rem;
        font-weight: 700;
        font-size: 0.8rem;
        cursor: pointer;
        transition: transform 0.2s;
    }
    .music-toggle:hover { transform: scale(1.05); }

    .visualizer {
        display: flex;
        gap: 3px;
        height: 12px;
    }
    .visualizer span {
        width: 3px;
        height: 100%;
        background: #000;
        border-radius: 1px;
    }
    .visualizer.playing span { animation: wave 0.6s infinite alternate; }
    .visualizer span:nth-child(2) { animation-delay: 0.2s; }
    .visualizer span:nth-child(3) { animation-delay: 0.4s; }

    @keyframes wave { to { height: 30%; } }
    @keyframes pulse { 
        0%, 100% { transform: scale(1); opacity: 1; }
        50% { transform: scale(1.1); opacity: 0.7; }
    }

    .notice { font-size: 0.7rem; color: #94a3b8; margin-top: 0.5rem; }

    @media (max-width: 600px) {
        .binary-tree { font-size: 0.8rem; }
        .info-card { width: 80vw; padding: 1.5rem; }
        .controls { right: 50%; transform: translateX(50%); bottom: 1rem; align-items: center; width: 100%; }
    }
</style>