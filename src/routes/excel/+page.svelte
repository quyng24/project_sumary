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

<BackHome />

<svelte:head>
	<title>Excel Mini Demo</title>
	<meta
		name="description"
		content="Mini spreadsheet demo với công thức cơ bản, công thức tham chiếu ô và giao diện nhập liệu dạng bảng."
	/>
</svelte:head>

<section class="excel-page">
	<header class="hero">
		<div class="hero-copy">
			<p class="eyebrow">SPREADSHEET DEMO</p>
			<h1>Mini Excel để nhập dữ liệu và thử công thức ngay trong trình duyệt.</h1>
			<p class="lede">
				Hỗ trợ nhập giá trị thường, tham chiếu ô và một số hàm cơ bản như `SUM`, `AVERAGE`,
				`MEDIAN`, `MIN`, `MAX`, `RANDOM`.
			</p>
		</div>

		<div class="formula-panel">
			<div class="formula-meta">
				<span class="meta-label">Cell</span>
				<strong>{activeCellId ?? '--'}</strong>
			</div>
			<div class="formula-input">
				<span class="meta-label">Formula</span>
				<code>{activeCellId ? (cellsMap.get(activeCellId)?.value ?? '') : ''}</code>
			</div>
		</div>
	</header>

	<div class="sheet-card">
		<div class="examples">
			<span>=SUM(A1:A5)</span>
			<span>=AVERAGE(B1,B2,B3)</span>
			<span>=A1*B1+10</span>
			<span>=RANDOM(1,100)</span>
		</div>

		<div class="sheet-scroll">
			<div class="spreadsheet">
				<div class="label corner"></div>
				{#each letters as letter, index (index)}
					<div class="label col-header">{letter}</div>
				{/each}

				{#each numbers as n, index (index)}
					<div class="label row-header">{n}</div>
					{#each letters as l, index (index)}
						{@const id = `${l}${n}`}
						{@const cell = cellsMap.get(id)}
						<div class="cell-wrapper">
							<input
								type="text"
								class:active={activeCellId === id}
								value={activeCellId === id ? (cell?.value ?? '') : (cell?.display ?? '')}
								placeholder={id}
								onfocus={() => (activeCellId = id)}
								onblur={() => (activeCellId = null)}
								onkeydown={(e) => e.key === 'Enter' && (e.target as HTMLInputElement).blur()}
								oninput={(e) => updateCell(id, e.currentTarget.value)}
							/>
						</div>
					{/each}
				{/each}
			</div>
		</div>
	</div>
</section>

<div class="toolbar">
	<div class="info">
		<b>Selected:</b> {activeCellId ?? '--'}
	</div>
	<div class="help-grid">
		<span>Range: `A1:A5`</span>
		<span>Refs: `=A1+B2`</span>
		<span>Press `Enter` to finish edit</span>
	</div>
</div>

<style>
	:global(body) {
		margin: 0;
		background:
			radial-gradient(circle at top left, rgba(34, 197, 94, 0.12), transparent 24%),
			radial-gradient(circle at top right, rgba(59, 130, 246, 0.12), transparent 26%),
			linear-gradient(180deg, #07111b 0%, #0b1623 100%);
		color: #f8fafc;
		font-family:
			'Inter',
			system-ui,
			sans-serif;
	}

	.excel-page {
		min-height: 100vh;
		box-sizing: border-box;
		padding: 6.5rem 1.25rem 5rem;
	}

	.hero {
		display: grid;
		grid-template-columns: minmax(0, 1.4fr) minmax(280px, 0.8fr);
		gap: 1.25rem;
		width: min(1180px, 100%);
		margin: 0 auto 1.25rem;
	}

	.hero-copy,
	.formula-panel,
	.sheet-card {
		border: 1px solid rgba(148, 163, 184, 0.14);
		border-radius: 24px;
		background: rgba(15, 23, 42, 0.72);
		backdrop-filter: blur(16px);
		box-shadow: 0 24px 60px rgba(0, 0, 0, 0.24);
	}

	.hero-copy {
		padding: 1.6rem;
	}

	.eyebrow {
		margin: 0 0 0.85rem;
		color: #86efac;
		font-size: 0.8rem;
		font-weight: 800;
		letter-spacing: 0.18em;
	}

	h1 {
		margin: 0;
		font-size: clamp(2rem, 5vw, 3.4rem);
		line-height: 1.02;
		letter-spacing: -0.05em;
	}

	.lede {
		margin: 1rem 0 0;
		max-width: 48rem;
		color: #cbd5e1;
		font-size: 1rem;
		line-height: 1.7;
	}

	.formula-panel {
		display: grid;
		gap: 1rem;
		padding: 1.25rem;
		align-content: start;
	}

	.formula-meta,
	.formula-input {
		display: grid;
		gap: 0.4rem;
	}

	.meta-label {
		color: #94a3b8;
		font-size: 0.78rem;
		font-weight: 700;
		letter-spacing: 0.1em;
		text-transform: uppercase;
	}

	.formula-panel strong,
	.formula-panel code {
		color: #f8fafc;
	}

	.formula-panel code {
		display: block;
		min-height: 2.5rem;
		padding: 0.85rem 1rem;
		border: 1px solid rgba(148, 163, 184, 0.14);
		border-radius: 16px;
		background: rgba(2, 6, 23, 0.55);
		font-family:
			'SFMono-Regular',
			Consolas,
			monospace;
		white-space: nowrap;
		overflow: auto;
	}

	.sheet-card {
		width: min(1180px, 100%);
		margin: 0 auto;
		padding: 1rem;
	}

	.examples {
		display: flex;
		flex-wrap: wrap;
		gap: 0.65rem;
		margin-bottom: 1rem;
	}

	.examples span {
		padding: 0.55rem 0.8rem;
		border: 1px solid rgba(148, 163, 184, 0.14);
		border-radius: 999px;
		background: rgba(30, 41, 59, 0.8);
		color: #cbd5e1;
		font-family:
			'SFMono-Regular',
			Consolas,
			monospace;
		font-size: 0.84rem;
	}

	.sheet-scroll {
		overflow: auto;
		max-height: calc(100vh - 19rem);
		border: 1px solid rgba(148, 163, 184, 0.12);
		border-radius: 18px;
		background: #0f172a;
	}

	.spreadsheet {
		display: grid;
		grid-template-columns: 56px repeat(10, 138px);
		width: max-content;
		min-width: 100%;
	}

	.label {
		position: sticky;
		display: flex;
		align-items: center;
		justify-content: center;
		height: 38px;
		border: 0.5px solid #334155;
		background: #162233;
		color: #94a3b8;
		font-size: 12px;
		font-weight: 800;
	}

	.col-header {
		top: 0;
		z-index: 10;
	}

	.row-header {
		left: 0;
		z-index: 10;
		width: 56px;
	}

	.corner {
		top: 0;
		left: 0;
		z-index: 20;
		background: #0b1220;
	}

	.cell-wrapper {
		height: 38px;
		border: 0.5px solid #334155;
		background: #132033;
	}

	input {
		width: 100%;
		height: 100%;
		border: none;
		padding: 0 10px;
		box-sizing: border-box;
		background: transparent;
		color: #e2e8f0;
		outline: none;
		font-size: 13px;
	}

	input::placeholder {
		color: #475569;
	}

	input.active {
		background: #0f172a;
		box-shadow: inset 0 0 0 2px #22c55e;
		color: #fff;
	}

	.toolbar {
		position: fixed;
		right: 0;
		bottom: 0;
		left: 0;
		display: flex;
		justify-content: space-between;
		gap: 1rem;
		padding: 0.85rem 1.25rem;
		border-top: 1px solid rgba(148, 163, 184, 0.14);
		background: rgba(15, 23, 42, 0.92);
		backdrop-filter: blur(14px);
		font-size: 13px;
		z-index: 100;
	}

	.help-grid {
		display: flex;
		flex-wrap: wrap;
		gap: 1rem;
		color: #94a3b8;
		font-family:
			'SFMono-Regular',
			Consolas,
			monospace;
	}

	@media (max-width: 900px) {
		.hero {
			grid-template-columns: 1fr;
		}

		.sheet-scroll {
			max-height: calc(100vh - 23rem);
		}
	}

	@media (max-width: 640px) {
		.excel-page {
			padding: 5.6rem 0.85rem 5.5rem;
		}

		.hero-copy,
		.formula-panel,
		.sheet-card {
			border-radius: 18px;
		}

		.sheet-card {
			padding: 0.8rem;
		}

		.toolbar {
			flex-direction: column;
			align-items: flex-start;
		}
	}
</style>
