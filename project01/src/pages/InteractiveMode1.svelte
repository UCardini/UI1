<script>
	import { getContext } from 'svelte';
	import StandbyMode from './StandbyMode.svelte';

	const profileState = getContext('profileState');

	function genHistory(profile) {
		const now        = Date.now();
		const days       = 90;
		const h          = [];
		let w            = profile.startWeight;
		const dailyDrift = (profile.currentWeight - profile.startWeight) / days;
		for (let i = days; i >= 1; i--) {
			const jitter = (Math.random() - 0.46) * 1.1;
			w += dailyDrift + jitter;
			h.push({ date: now - i * 86_400_000, weight: parseFloat(w.toFixed(1)) });
		}
		return h;
	}

	let BASE_HISTORY = genHistory(profileState.active);

	let holding    = $state(false);
	let progress   = $state(0);
	let liveWeight = $state(null);
	let weight     = $state(null);
	let history    = $state([...BASE_HISTORY]);
	let stats      = $state({ weekly: '0.0', total: '0.0' });
	let _timer     = null;

	const R    = 36;
	const CIRC = 2 * Math.PI * R;
	let dashOffset = $derived(CIRC * (1 - progress));

	function greeting() {
		const h    = new Date().getHours();
		const name = profileState.active.name;
		if (h < 12) return `Good morning, ${name}!`;
		if (h < 17) return `Good afternoon, ${name}!`;
		return `Good evening, ${name}!`;
	}

	function fmt(n) { return parseFloat(n) >= 0 ? `+${n}` : `${n}`; }

	function startHold(e, profile) {
		e.preventDefault();
		if (weight !== null) return;
		profileState.active = profile;
		BASE_HISTORY = genHistory(profile);
		history = [...BASE_HISTORY];
		holding = true; progress = 0; liveWeight = null;
		const start = Date.now();
		_timer = setInterval(() => {
			const elapsed = Date.now() - start;
			progress = Math.min(1, elapsed / 3000);
			if (elapsed > 400)
				liveWeight = (profile.currentWeight + (Math.random() - 0.5) * 1.6).toFixed(1);
		}, 120);
	}

	function endHold() {
		if (!holding) return;
		clearInterval(_timer);
		holding = false;
		if (progress >= 1 && liveWeight !== null) {
			const locked = liveWeight;
			liveWeight = null;
			weight = locked;

			const h = [...BASE_HISTORY, { date: Date.now(), weight: parseFloat(locked) }];
			history = h;

			const current = parseFloat(locked);
			const weekRef = [...h].filter(e => e.date <= Date.now() - 7 * 86_400_000).pop() ?? h[0];
			stats = {
				weekly: (current - weekRef.weight).toFixed(1),
				total:  (current - h[0].weight).toFixed(1),
			};
			setTimeout(() => { weight = null; progress = 0; history = [...BASE_HISTORY]; }, 12_000);
		} else {
			liveWeight = null; progress = 0;
		}
	}

	// ── Graph ─────────────────────────────────────────────────────
	const GW = 300, GH = 80;

	function graphData(hist) {
		const pts = hist.slice(-30);
		if (pts.length < 2) return null;
		const ws   = pts.map(p => p.weight);
		const minW = Math.min(...ws) - 1;
		const maxW = Math.max(...ws) + 1;
		const tx = i => (i / (pts.length - 1)) * GW;
		const ty = w => GH - ((w - minW) / (maxW - minW)) * GH * 0.82 - GH * 0.09;
		const coords = pts.map((p, i) => ({ x: tx(i), y: ty(p.weight) }));

		let line = `M ${coords[0].x} ${coords[0].y}`;
		for (let i = 1; i < coords.length; i++) {
			const p = coords[i - 1], c = coords[i];
			const mx = (p.x + c.x) / 2;
			line += ` C ${mx} ${p.y} ${mx} ${c.y} ${c.x} ${c.y}`;
		}
		const area = `${line} L ${coords.at(-1).x} ${GH} L ${coords[0].x} ${GH} Z`;
		const gridYs = [0.2, 0.5, 0.8].map(f => GH * f);
		return { line, area, coords, gridYs };
	}

	let graph = $derived(graphData(history));
</script>

<div class="wrapper">
	<StandbyMode />

	<!-- svelte-ignore a11y_no_static_element_interactions -->
	<div
		class="scale"
		class:full={weight !== null}
		onmouseup={endHold}
		onmouseleave={endHold}
		ontouchend={endHold}
	>
		<!-- Weight always top-left -->
		{#if liveWeight !== null || weight !== null}
			<div class="topleft">
				<span class="val" class:dim={weight === null}>{weight ?? liveWeight}</span>
				<span class="unit">lbs</span>
			</div>
		{/if}

		{#if weight !== null}
			<div class="greet-row">
				<span class="greet">{greeting()}</span>
			</div>

			<div class="stats-row">
				<div class="stat">
					<span class="stat-label">Weekly</span>
					<span class="stat-num" class:loss={parseFloat(stats.weekly) < 0} class:gain={parseFloat(stats.weekly) > 0}>
						{fmt(stats.weekly)} lbs
					</span>
				</div>
				<div class="stat-divider"></div>
				<div class="stat">
					<span class="stat-label">Total Change</span>
					<span class="stat-num" class:loss={parseFloat(stats.total) < 0} class:gain={parseFloat(stats.total) > 0}>
						{fmt(stats.total)} lbs
					</span>
				</div>
			</div>

			<div class="graph-area">
				{#if graph}
					<svg viewBox="0 0 {GW} {GH}" class="graph" preserveAspectRatio="xMidYMid meet">
						<defs>
							<linearGradient id="gfill" x1="0" y1="0" x2="0" y2="1">
								<stop offset="0%"   stop-color="var(--c-graph)" stop-opacity="0.22" />
								<stop offset="100%" stop-color="var(--c-graph)" stop-opacity="0" />
							</linearGradient>
						</defs>
						{#each graph.gridYs as gy}
							<line x1="0" y1={gy} x2={GW} y2={gy} stroke="var(--c-grid)" stroke-width="0.7" />
						{/each}
						<path d={graph.area} fill="url(#gfill)" />
						<path d={graph.line} fill="none" stroke="var(--c-graph)" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" />
						{#each graph.coords as c, i}
							{#if i === graph.coords.length - 1}
								<circle cx={c.x} cy={c.y} r="4" fill="var(--c-graph)" />
								<circle cx={c.x} cy={c.y} r="8" fill="var(--c-graph)" opacity="0.18" />
							{:else if i % 4 === 0}
								<circle cx={c.x} cy={c.y} r="2" fill="var(--c-graph)" opacity="0.45" />
							{/if}
						{/each}
					</svg>
				{/if}
			</div>

		{:else}
			<div class="center">
				<svg class="ring" viewBox="0 0 100 100">
					<circle cx="50" cy="50" r={R} fill="none" stroke="rgba(255,255,255,0.25)" stroke-width="5" />
					{#if holding}
						<circle cx="50" cy="50" r={R} fill="none" stroke="rgba(255,255,255,0.9)" stroke-width="5"
							stroke-dasharray={CIRC} stroke-dashoffset={dashOffset}
							transform="rotate(-90 50 50)" stroke-linecap="round"
						/>
					{/if}
				</svg>
				<p class="prompt">
					{holding ? 'Hold…' : 'Hold a name below to simulate stepping on the panel'}
				</p>
				<!-- svelte-ignore a11y_no_static_element_interactions -->
				<div class="names">
					{#each profileState.list as profile}
						<div
							class="name-zone"
							class:name-holding={holding && profileState.active.id === profile.id}
							onmousedown={(e) => startHold(e, profile)}
							ontouchstart={(e) => startHold(e, profile)}
						>
							{profile.name}
						</div>
					{/each}
				</div>
			</div>
		{/if}
	</div>
</div>

<style>
	.wrapper { position: relative; width: 100%; height: 100%; }

	.scale {
		--c-text:  rgba(255,255,255,0.92);
		--c-dim:   rgba(255,255,255,0.45);
		--c-sep:   rgba(255,255,255,0.18);
		--c-graph: rgba(255,255,255,0.8);
		--c-grid:  rgba(255,255,255,0.12);

		position: absolute;
		left: 0; right: 0; bottom: 0;
		height: 75%;
		background: rgba(255,255,255,0.13);
		backdrop-filter: blur(8px);
		-webkit-backdrop-filter: blur(8px);
		border-top: 1px solid rgba(255,255,255,0.3);
		border-radius: 12px 12px 0 0;
		user-select: none;
		transition: height 0.45s ease, border-radius 0.45s ease, background 0.45s ease;
	}

	.scale.full {
		--c-text:  #1a1a2e;
		--c-dim:   rgba(0,0,0,0.35);
		--c-sep:   rgba(0,0,0,0.08);
		--c-graph: #2563eb;
		--c-grid:  rgba(0,0,0,0.07);

		height: 100%;
		border-radius: 0;
		border-top: none;
		background: rgba(255,255,255,0.55);
		backdrop-filter: blur(18px);
		-webkit-backdrop-filter: blur(18px);
		cursor: default;
	}

	.topleft {
		position: absolute;
		top: 14px; left: 16px;
		display: flex;
		align-items: baseline;
		gap: 5px;
	}

	.val {
		font-family: monospace;
		font-size: 48px;
		font-weight: bold;
		color: var(--c-text);
		line-height: 1;
		transition: color 0.2s;
	}

	.val.dim { color: var(--c-dim); }

	.unit { font-family: sans-serif; font-size: 16px; color: var(--c-dim); }

	.greet-row {
		position: absolute;
		top: 26px; right: 16px;
		text-align: right;
	}

	.greet {
		font-family: sans-serif;
		font-size: 18px;
		font-weight: 300;
		color: var(--c-text);
	}

	.stats-row {
		position: absolute;
		top: 80px; left: 0; right: 0;
		display: flex;
		align-items: center;
		padding: 10px 18px;
		border-top: 1px solid var(--c-sep);
		border-bottom: 1px solid var(--c-sep);
		box-sizing: border-box;
	}

	.stat { flex: 1; display: flex; flex-direction: column; gap: 2px; }

	.stat-label {
		font-family: sans-serif;
		font-size: 9px;
		text-transform: uppercase;
		letter-spacing: 0.08em;
		color: var(--c-dim);
	}

	.stat-num {
		font-family: monospace;
		font-size: 22px;
		font-weight: bold;
		color: var(--c-text);
	}

	.stat-num.loss { color: #16a34a; }
	.stat-num.gain { color: #dc2626; }

	.stat-divider {
		width: 1px; height: 32px;
		background: var(--c-sep);
		margin: 0 16px;
		flex-shrink: 0;
	}

	.graph-area {
		position: absolute;
		top: 158px; left: 12px; right: 12px; bottom: 12px;
		display: flex;
		align-items: stretch;
	}

	.graph { width: 100%; height: 100%; }

	.center {
		position: absolute;
		inset: 0;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		gap: 12px;
	}

	.ring { width: 72px; height: 72px; flex-shrink: 0; }

	.prompt {
		color: rgba(255,255,255,0.7);
		font-family: sans-serif;
		font-size: 12px;
		text-align: center;
		max-width: 200px;
		line-height: 1.4;
		margin: 0;
	}

	/* ── Name zones ─────────────────────────── */
	.names {
		display: flex;
		align-items: center;
		gap: 0;
		margin-top: 8px;
	}

	.name-zone {
		padding: 8px 18px;
		font-family: sans-serif;
		font-size: 14px;
		font-weight: 500;
		color: rgba(255,255,255,0.6);
		cursor: pointer;
		transition: color 0.15s;
		border-right: 1px solid rgba(255,255,255,0.2);
	}

	.name-zone:last-child { border-right: none; }

	.name-zone:hover          { color: rgba(255,255,255,0.9); }
	.name-zone.name-holding   { color: #fff; font-weight: 700; }
</style>
