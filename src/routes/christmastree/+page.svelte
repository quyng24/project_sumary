<script lang="ts">
    import { onMount } from 'svelte';
    import merryChristmasTrack from '$lib/assets/merry-christmas.mp3';
	import BackHome from '$lib/components/BackHome.svelte';

    const TREE_HEIGHT = 15;
    const SNOWFLAKE_INTERVAL_MS  = 150;
    const TREE_REFRESH_INTERVAL_MS = 500;
    const currentYear = new Date().getFullYear();
    const yearDigits = String(currentYear).split('');

    interface TreeCell {
        digit: '0' | '1';
        color: string;
        glow: boolean;
    }

    interface Snowflake {
        id: number;
        left: string;
        duration: string;
        size: string;
        blur: string;
    }

    let treeRows   = $state<TreeCell[][]>([]);
    let snowflakes = $state<Snowflake[]>([]);
    let isPlaying  = $state(false);
    let audioNotice = $state('Nhấn PLAY MUSIC để bật nhạc nền.');

    let snowId = 0;
    let audioEl: HTMLAudioElement | null = null;

    function generateTree(): void {
        const rows: TreeCell[][] = [];
        for (let i = 0; i < TREE_HEIGHT; i++) {
            const row: TreeCell[] = [];
            for (let j = 0; j < 2 * i + 1; j++) {
                const rand = Math.random();
                const isOrnament = rand < 0.15;
                row.push({
                    digit: Math.random() < 0.5 ? '0' : '1',
                    color: isOrnament ? (rand < 0.07 ? '#ff0055' : '#ffcc00') : '#2ecc71',
                    glow: isOrnament
                });
            }
            rows.push(row);
        }
        treeRows = rows;
    }

    function spawnSnowflake(): void {
        const flake: Snowflake = {
            id:       ++snowId,
            left:     `${Math.random() * 100}vw`,
            duration: `${Math.random() * 5 + 5}s`, // Rơi chậm hơn cho lãng mạn
            size:     `${Math.random() * 10 + 5}px`,
            blur:     `${Math.random() * 2}px`
        };
        snowflakes = [...snowflakes, flake];

        setTimeout(() => {
            snowflakes = snowflakes.filter(s => s.id !== flake.id);
        }, 10000);
    }

    async function toggleAudio(): Promise<void> {
        if (!audioEl) return;
        if (isPlaying) {
            audioEl.pause();
            isPlaying = false;
            audioNotice = 'Nhạc nền đang tạm dừng.';
        } else {
            try {
                await audioEl.play();
                isPlaying = true;
                audioNotice = 'Đang phát Merry Christmas.';
            } catch {
                isPlaying = false;
                audioNotice = 'Trình duyệt chặn autoplay. Hãy nhấn nút thêm một lần.';
            }
        }
    }

    onMount(() => {
        if (audioEl) {
            audioEl.volume = 0.55;
        }

        generateTree();
        const treeTimer  = setInterval(generateTree, TREE_REFRESH_INTERVAL_MS);
        const snowTimer  = setInterval(spawnSnowflake, SNOWFLAKE_INTERVAL_MS);

        return () => {
            clearInterval(treeTimer);
            clearInterval(snowTimer);
        };
    });

    function spaces(rowIndex: number): number {
        return TREE_HEIGHT - rowIndex - 1;
    }
</script>

<div class="snowfall" aria-hidden="true">
    <BackHome/>
    {#each snowflakes as flake (flake.id)}
        <span
            class="snowflake"
            style="left:{flake.left}; animation-duration:{flake.duration}; font-size:{flake.size}; filter: blur({flake.blur});"
        >❄</span>
    {/each}
</div>

<button class="play-btn" class:playing={isPlaying} onclick={toggleAudio}>
    <span class="icon">{isPlaying ? '⏸' : '▶'}</span>
    {isPlaying ? 'PAUSE' : 'PLAY MUSIC'}
</button>

<audio bind:this={audioEl} loop preload="auto">
    <source src={merryChristmasTrack} type="audio/mpeg" />
</audio>

<p class="audio-notice">{audioNotice}</p>

<div class="main-wrapper">
    <div class="tree-container">
        <div class="star">★</div>

        <div class="tree">
        {#each treeRows as row, i (i)}
            <div class="tree-row">
                <span class="sp">{" ".repeat(spaces(i))}</span>
                
                {#each row as cell, j (j)}
                    <span 
                        class:ornament={cell.glow} 
                        style="color:{cell.color}"
                    >{cell.digit}</span>
                {/each}
            </div>
        {/each}
</div>

        <div class="trunk" aria-hidden="true">
            <span class="trunk-cap"></span>
            <span class="trunk-core"></span>
            <span class="trunk-shadow"></span>
        </div>
    </div>

    <div class="glass-card">
        <div class="bangrol">
            <p class="merry">Merry Christmas</p>
            <div class="year" aria-label={`Year ${currentYear}`}>
                {#each yearDigits as digit, index (index)}
                    <span class="year-digit" style="--digit-index:{index}">{digit}</span>
                {/each}
            </div>
        </div>

        <div class="name">
            <p>Nguyen Quy Coder</p>
        </div>
    </div>
</div>

<style>
    :global(body) {
        margin: 0;
        padding: 0;
        display: flex;
        justify-content: center;
        align-items: center;
        min-height: 100vh;
        background: radial-gradient(circle at center, #1a2a6c, #b21f1f, #fdbb2d);
        background-size: 400% 400%;
        animation: gradientBG 15s ease infinite;
        color: white;
        font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        overflow: hidden;
    }

    @keyframes gradientBG {
        0% { background-position: 0% 50%; }
        50% { background-position: 100% 50%; }
        100% { background-position: 0% 50%; }
    }

    .main-wrapper {
        display: flex;
        flex-direction: column;
        align-items: center;
        gap: 20px;
        z-index: 1;
    }

    .audio-notice {
        position: fixed;
        top: 84px;
        right: 30px;
        margin: 0;
        max-width: 260px;
        padding: 10px 14px;
        border-radius: 16px;
        background: rgba(7, 12, 26, 0.35);
        color: rgba(255, 255, 255, 0.8);
        font-size: 0.82rem;
        line-height: 1.45;
        backdrop-filter: blur(10px);
        z-index: 100;
    }

    /* ── Snowfall ── */
    .snowflake {
        position: absolute;
        top: -20px;
        color: white;
        opacity: 0.6;
        user-select: none;
        animation: fall linear forwards;
    }

    @keyframes fall {
        0%   { transform: translateY(0) rotate(0deg); }
        100% { transform: translateY(110vh) rotate(360deg); }
    }

    /* ── Play button ── */
    .play-btn {
        position: fixed;
        top: 30px;
        right: 30px;
        background: rgba(255, 255, 255, 0.15);
        backdrop-filter: blur(10px);
        color: white;
        padding: 12px 24px;
        border: 1px solid rgba(255, 255, 255, 0.2);
        border-radius: 50px;
        cursor: pointer;
        font-weight: 800;
        letter-spacing: 1px;
        z-index: 100;
        transition: all 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
        display: flex;
        align-items: center;
        gap: 10px;
    }

    .play-btn:hover {
        background: rgba(255, 255, 255, 0.25);
        transform: scale(1.05);
    }

    .play-btn.playing {
        border-color: #2ecc71;
        box-shadow: 0 0 15px rgba(46, 204, 113, 0.4);
    }

    /* ── Tree ── */
    .star {
        color: #f1c40f;
        font-size: 60px;
        filter: drop-shadow(0 0 20px #f1c40f);
        animation: starPulse 1.5s infinite ease-in-out;
        line-height: 1;
    }

    @keyframes starPulse {
        0%, 100% { transform: scale(1) rotate(0); filter: drop-shadow(0 0 15px #f1c40f); }
        50% { transform: scale(1.2) rotate(15deg); filter: drop-shadow(0 0 30px #f1c40f); }
    }

    .tree-container {
        display: flex;
        flex-direction: column;
        align-items: center;
        animation: treeSway 5.8s ease-in-out infinite;
        transform-origin: center bottom;
    }

    .tree {
        font-family: 'Courier New', Courier, monospace;
        font-size: 1.2rem;
        font-weight: bold;
        line-height: 1.2;
        text-shadow: 0 0 5px rgba(0,0,0,0.3);
    }

    .tree-row {
        white-space: pre;
    }

    .ornament {
        text-shadow: 0 0 10px currentColor;
        font-weight: 900;
    }

    .trunk {
        position: relative;
        width: 92px;
        height: 112px;
        margin-top: 10px;
        display: flex;
        justify-content: center;
        align-items: flex-start;
    }

    .trunk-cap,
    .trunk-core,
    .trunk-shadow {
        position: absolute;
        border-radius: 18px;
    }

    .trunk-cap {
        top: 0;
        width: 56px;
        height: 18px;
        background: linear-gradient(180deg, #9b6a42 0%, #754521 100%);
        box-shadow: 0 8px 20px rgba(0,0,0,0.25);
    }

    .trunk-core {
        top: 10px;
        width: 72px;
        height: 86px;
        overflow: hidden;
        background:
            linear-gradient(90deg, rgba(255,255,255,0.18), transparent 20%, transparent 80%, rgba(0,0,0,0.22)),
            repeating-linear-gradient(
                90deg,
                #7d4d2f 0 10px,
                #8f5d3a 10px 20px,
                #6d3f24 20px 30px
            );
        box-shadow:
            inset 0 0 0 1px rgba(255,255,255,0.06),
            inset 0 -8px 14px rgba(57, 29, 14, 0.26),
            0 18px 30px rgba(0,0,0,0.3);
        animation: trunkGlow 4.5s ease-in-out infinite;
    }

    .trunk-core::before {
        content: '';
        position: absolute;
        inset: -20% 55% -20% -15%;
        background: linear-gradient(110deg, transparent 0%, rgba(255,255,255,0.18) 40%, transparent 72%);
        animation: barkShimmer 3.8s linear infinite;
    }

    .trunk-shadow {
        bottom: 0;
        width: 92px;
        height: 20px;
        border-radius: 50%;
        background: radial-gradient(circle, rgba(0,0,0,0.42) 0%, rgba(0,0,0,0) 72%);
    }

    /* ── Glass Card ── */
    .glass-card {
        background: rgba(255, 255, 255, 0.1);
        backdrop-filter: blur(15px);
        padding: 30px 50px;
        border-radius: 24px;
        border: 1px solid rgba(255, 255, 255, 0.1);
        box-shadow: 0 20px 40px rgba(0,0,0,0.2);
        text-align: center;
    }

    .merry {
        font-size: 40px;
        font-family: 'Pacifico', cursive, serif;
        margin-bottom: 5px;
        background: linear-gradient(to right, #ffefba, #ffffff);
        background-clip: text;
        -webkit-background-clip: text;
        -webkit-text-fill-color: transparent;
    }

    .year {
        display: flex;
        justify-content: center;
        gap: 0.55rem;
        margin-bottom: 15px;
    }

    .year-digit {
        display: inline-flex;
        align-items: center;
        justify-content: center;
        min-width: 2.2rem;
        padding: 0.45rem 0.2rem;
        border-radius: 14px;
        background: rgba(255, 204, 0, 0.12);
        border: 1px solid rgba(255, 204, 0, 0.18);
        color: #ffcc00;
        font-size: 1.6rem;
        font-weight: 800;
        letter-spacing: 0.06em;
        box-shadow: 0 0 18px rgba(255, 204, 0, 0.12);
        animation: yearPulse 2.1s ease-in-out infinite;
        animation-delay: calc(var(--digit-index) * 0.18s);
    }

    .name {
        font-size: 1.2rem;
        letter-spacing: 2px;
        text-transform: uppercase;
        color: rgba(255, 255, 255, 0.7);
        border-top: 1px solid rgba(255,255,255,0.1);
        padding-top: 15px;
    }

    .sp { white-space: pre; }

    @keyframes treeSway {
        0%, 100% { transform: rotate(-1.5deg); }
        50% { transform: rotate(1.5deg); }
    }

    @keyframes trunkGlow {
        0%, 100% {
            transform: translateY(0);
            box-shadow:
                inset 0 0 0 1px rgba(255,255,255,0.06),
                inset 0 -8px 14px rgba(57, 29, 14, 0.26),
                0 18px 30px rgba(0,0,0,0.3);
        }
        50% {
            transform: translateY(2px);
            box-shadow:
                inset 0 0 0 1px rgba(255,255,255,0.1),
                inset 0 -10px 18px rgba(57, 29, 14, 0.34),
                0 24px 34px rgba(0,0,0,0.34);
        }
    }

    @keyframes barkShimmer {
        0% { transform: translateX(-50%) rotate(8deg); }
        100% { transform: translateX(120%) rotate(8deg); }
    }

    @keyframes yearPulse {
        0%, 100% {
            transform: translateY(0) scale(1);
            box-shadow: 0 0 18px rgba(255, 204, 0, 0.12);
        }
        50% {
            transform: translateY(-4px) scale(1.06);
            box-shadow: 0 0 24px rgba(255, 204, 0, 0.28);
        }
    }

    /* ── Responsive ── */
    @media (max-width: 600px) {
        .tree { font-size: 0.9rem; }
        .glass-card { padding: 20px; width: 90vw; }
        .merry { font-size: 28px; }
        .play-btn { top: 20px; right: 20px; padding: 8px 16px; }
        .audio-notice { top: 68px; right: 20px; max-width: 220px; font-size: 0.76rem; }
        .trunk { width: 78px; height: 100px; }
        .trunk-cap { width: 48px; }
        .trunk-core { width: 62px; height: 78px; }
        .year-digit { min-width: 1.8rem; font-size: 1.3rem; }
    }
</style>
