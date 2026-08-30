<svelte:options runes={true} />
<script lang="ts">
	import BackHome from '$lib/components/BackHome.svelte';
	import { SvelteMap } from 'svelte/reactivity';

	const letters = ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J'];
	const numbers = Array.from({ length: 50 }, (_, i) => i + 1);

	type CellData = {
		value: string;
		display: string;
	};

	let cellsMap = $state(new SvelteMap<string, CellData>());
	let activeCellId = $state<string | null>(null);

	const spreadsheetFunctions: Record<string, (nums: number[]) => number> = {
		sum: (nums) => nums.reduce((a, b) => a + b, 0),
		average: (nums) => (nums.length ? nums.reduce((a, b) => a + b, 0) / nums.length : 0),
		median: (nums) => {
			const s = [...nums].sort((a, b) => a - b);
			const mid = Math.floor(s.length / 2);
			return s.length ? (s.length % 2 !== 0 ? s[mid] : (s[mid - 1] + s[mid]) / 2) : 0;
		},
		random: ([min, max]) => Math.floor(Math.random() * (max - min + 1) + min),
		max: (nums) => (nums.length ? Math.max(...nums) : 0),
		min: (nums) => (nums.length ? Math.min(...nums) : 0)
	};

	const getCellValue = (id: string): number => {
		const cell = cellsMap.get(id.toUpperCase());
		if (!cell) return 0;
		const val = parseFloat(cell.display);
		return isNaN(val) ? 0 : val;
	};

	const parseAndEval = (formula: string): string => {
		try {
			let processed = formula.toUpperCase();

			processed = processed.replace(/([A-J])(\d+):([A-J])(\d+)/g, (_, c1, n1, c2, n2) => {
				const res = [];
				for (let n = parseInt(n1); n <= parseInt(n2); n++) {
					for (let c = c1.charCodeAt(0); c <= c2.charCodeAt(0); c++) {
						res.push(String.fromCharCode(c) + n);
					}
				}
				return res.join(',');
			});

			processed = processed.replace(/[A-J]\d+/g, (id) => String(getCellValue(id)));

			processed = processed.replace(/([A-Z]+)\(([^()]+)\)/g, (_, fn, args) => {
				const func = spreadsheetFunctions[fn.toLowerCase()];
				if (func) {
					const nums = args
						.split(',')
						.map((a: string) => parseFloat(a.trim()))
						.filter((n: number) => !isNaN(n));
					return String(func(nums));
				}
				return '0';
			});

			return String(Function(`"use strict"; return (${processed})`)());
		} catch {
			return '#ERROR!';
		}
	};

	function updateCell(id: string, val: string) {
		const nextMap = new SvelteMap(cellsMap);
		const newData = { value: val, display: '' };

		if (val.startsWith('=')) {
			newData.display = parseAndEval(val.slice(1));
		} else {
			newData.display = val;
		}

		nextMap.set(id, newData);

		nextMap.forEach((cell, key) => {
			if (cell.value.startsWith('=') && key !== id) {
				nextMap.set(key, {
					...cell,
					display: parseAndEval(cell.value.slice(1))
				});
			}
		});

		cellsMap = nextMap;
	}
</script>

<div class="excel-page-container">
    
    <header class="app-header">
        <div class="header-left">
            <BackHome />
        </div>
        <div class="header-center">
            <div class="file-name">
                <span class="icon">📊</span>
                <h1>Svelte_Spreadsheet_2024.xlsx</h1>
                <span class="tag">Auto-save on</span>
            </div>
        </div>
        <div class="header-right">
            <div class="user-badge">QC</div>
        </div>
    </header>

    <section class="formula-bar">
        <div class="active-cell-display">{activeCellId ?? ''}</div>
        <div class="fx-label">ƒx</div>
        <input 
            type="text" 
            readonly
            placeholder="Select a cell or enter formula starting with ="
            value={activeCellId ? (cellsMap.get(activeCellId)?.value ?? '') : ''} 
        />
    </section>

    <main class="sheet-viewport">
        <div class="grid-table">
            <div class="label corner"></div>
            
            {#each letters as letter, i(i)}
                <div class="label col-header">{letter}</div>
            {/each}

            {#each numbers as n, i (i)}
                <div class="label row-header">{n}</div>
                {#each letters as l, i(i)}
                    {@const id = `${l}${n}`}
                    {@const cell = cellsMap.get(id)}
                    <div class="cell-unit" class:is-active={activeCellId === id}>
                        <input
                            type="text"
                            value={activeCellId === id ? (cell?.value ?? '') : (cell?.display ?? '')}
                            onfocus={() => (activeCellId = id)}
                            onblur={() => (activeCellId = null)}
                            onkeydown={(e) => e.key === 'Enter' && (e.target as HTMLInputElement).blur()}
                            oninput={(e) => updateCell(id, e.currentTarget.value)}
                        />
                    </div>
                {/each}
            {/each}
        </div>
    </main>

    <footer class="footer-bar">
        <div class="status">● System Ready</div>
        <div class="zoom-level">100%</div>
    </footer>
</div>

<style>
    .excel-page-container {
        --bg-main: #09090b;
        --bg-side: #18181b;
        --border: #27272a;
        --accent: #10b981;
        --text: #e4e4e7;
        --text-dim: #71717a;

        position: fixed;
        inset: 0;
        display: flex;
        flex-direction: column;
        background-color: var(--bg-main);
        color: var(--text);
        font-family: 'Inter', system-ui, sans-serif;
    }

    .app-header {
        height: 56px;
        display: flex;
        align-items: center;
        justify-content: space-between;
        padding: 0 1rem;
        background: var(--bg-side);
        border-bottom: 1px solid var(--border);
    }

    .header-left {
        min-width: 120px;
        padding-left: 0.5rem; 
    }

    .header-center {
        flex: 1;
        display: flex;
        justify-content: center;
    }

    .file-name {
        display: flex;
        align-items: center;
        gap: 0.75rem;
        background: #00000040;
        padding: 0.4rem 1rem;
        border-radius: 8px;
        border: 1px solid var(--border);
    }

    .file-name h1 { font-size: 0.85rem; font-weight: 500; margin: 0; color: var(--text); }
    .tag { font-size: 0.6rem; color: var(--accent); font-weight: 700; text-transform: uppercase; }

    .user-badge {
        width: 32px;
        height: 32px;
        background: var(--accent);
        color: #064e3b;
        border-radius: 50%;
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 0.75rem;
        font-weight: 700;
    }

    .formula-bar {
        height: 38px;
        display: flex;
        align-items: center;
        background: var(--bg-side);
        border-bottom: 1px solid var(--border);
    }

    .active-cell-display {
        width: 70px;
        text-align: center;
        font-family: monospace;
        color: var(--accent);
        font-weight: 600;
        border-right: 1px solid var(--border);
    }

    .fx-label {
        width: 40px;
        text-align: center;
        font-style: italic;
        color: var(--text-dim);
        font-family: serif;
        font-size: 1.1rem;
    }

    .formula-bar input {
        flex: 1;
        background: transparent;
        border: none;
        color: var(--text);
        padding: 0 1rem;
        font-size: 0.85rem;
        outline: none;
        font-family: 'JetBrains Mono', monospace;
    }

    .sheet-viewport {
        flex: 1;
        overflow: auto;
        background: var(--bg-main);
    }

    .grid-table {
        display: grid;
        grid-template-columns: 50px repeat(10, 130px);
        width: max-content;
    }

    .label {
        background: #111113;
        color: var(--text-dim);
        font-size: 0.7rem;
        font-weight: 600;
        display: flex;
        align-items: center;
        justify-content: center;
        border-right: 1px solid var(--border);
        border-bottom: 1px solid var(--border);
        position: sticky;
        user-select: none;
    }

    .col-header { top: 0; height: 32px; z-index: 10; }
    .row-header { left: 0; width: 50px; z-index: 10; }
    .corner { top: 0; left: 0; z-index: 20; background: #27272a; }

    .cell-unit {
        height: 32px;
        border-right: 1px solid var(--border);
        border-bottom: 1px solid var(--border);
        background: transparent;
        transition: background 0.1s;
    }

    .cell-unit.is-active {
        box-shadow: inset 0 0 0 2px var(--accent);
        z-index: 5;
        background: #10b98105;
    }

    .cell-unit input {
        width: 100%;
        height: 100%;
        border: none;
        background: transparent;
        color: #fff;
        padding: 0 10px;
        font-size: 0.85rem;
        outline: none;
    }

    .footer-bar {
        height: 24px;
        background: var(--accent);
        color: #064e3b;
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 0 1rem;
        font-size: 0.65rem;
        font-weight: 800;
    }

    /* Scrollbar */
    .sheet-viewport::-webkit-scrollbar { width: 12px; height: 12px; }
    .sheet-viewport::-webkit-scrollbar-track { background: var(--bg-main); }
    .sheet-viewport::-webkit-scrollbar-thumb { background: #3f3f46; border: 3px solid var(--bg-main); border-radius: 10px; }
</style>
