<script>
	import { getContext } from 'svelte';
	import WidgetContent from '../lib/WidgetContent.svelte';

	const standbyConfig = getContext('standbyConfig');

	const WIDGET_TYPES = [
		{ type: 'clock',     label: 'Clock'     },
		{ type: 'clock2',    label: 'Analog Clock' },
		{ type: 'todo',      label: 'To-Do'     },
		{ type: 'date',      label: 'Date'      },
		{ type: 'calendar',  label: 'Calendar'  },
		{ type: 'occupancy', label: 'Occupancy' },
		{ type: 'news',      label: 'News'      },
		{ type: 'weather',   label: 'Weather'   },
	];

	const DEFAULT_COLOR = '#3a3a3a';
	const PALETTE = [
		'#3a3a3a', // grey
		'#c0392b', // red
		'#e67e22', // orange
		'#f1c40f', // yellow
		'#27ae60', // green
		'#2980b9', // blue
		'#8e44ad', // purple
		'#e91e8c', // pink
	];

	let pageEl;
	let selected  = $state(null); // widget id or null
	let menuOpen  = $state(false);
	let dragType  = $state(null);

	let _drag     = null;
	let _justDragged = false;
	let _nextId   = Math.max(0, ...standbyConfig.widgets.map(w => w.id));

	// ── helpers ─────────────────────────────────────────────────
	function cellCoords(e) {
		const r = pageEl.getBoundingClientRect();
		return {
			col: (e.clientX - r.left) / (r.width  / 8),
			row: (e.clientY - r.top)  / (r.height / 8),
		};
	}

	function overlaps(excludeId, c, r, w, h) {
		return standbyConfig.widgets.some(x =>
			x.id !== excludeId &&
			c < x.colStart + x.wSteps && c + w > x.colStart &&
			r < x.rowStart + x.hSteps && r + h > x.rowStart
		);
	}

	function findFreeSpot(wSteps = 2, hSteps = 2) {
		for (let r = 0; r <= 8 - hSteps; r++)
			for (let c = 0; c <= 8 - wSteps; c++)
				if (!overlaps(-1, c, r, wSteps, hSteps))
					return { colStart: c, rowStart: r };
		return { colStart: 0, rowStart: 0 };
	}

	// ── drag ────────────────────────────────────────────────────
	function startDrag(type, widgetId, e) {
		const w = standbyConfig.widgets.find(x => x.id === widgetId);
		const { col, row } = cellCoords(e);
		_drag = {
			type, widgetId,
			grabCol: col - w.colStart,
			grabRow: row - w.rowStart,
			moved: false,
		};
		dragType = type;
		window.addEventListener('mousemove', onMove);
		window.addEventListener('mouseup',   onUp);
	}

	function onMove(e) {
		if (!_drag) return;
		const w = standbyConfig.widgets.find(x => x.id === _drag.widgetId);
		if (!w) return;
		const { col, row } = cellCoords(e);
		_drag.moved = true;

		if (_drag.type === 'move') {
			const c = Math.max(0, Math.min(8 - w.wSteps, Math.round(col - _drag.grabCol)));
			const r = Math.max(0, Math.min(8 - w.hSteps, Math.round(row - _drag.grabRow)));
			if (!overlaps(w.id, c, r, w.wSteps, w.hSteps)) {
				w.colStart = c;
				w.rowStart = r;
			}
		} else {
			const nw = Math.max(1, Math.min(8 - w.colStart, Math.round(col - w.colStart)));
			const nh = Math.max(1, Math.min(8 - w.rowStart, Math.round(row - w.rowStart)));
			if (!overlaps(w.id, w.colStart, w.rowStart, nw, nh)) {
				w.wSteps = nw;
				w.hSteps = nh;
			}
		}
	}

	function onUp() {
		_justDragged = !!_drag?.moved;
		_drag    = null;
		dragType = null;
		window.removeEventListener('mousemove', onMove);
		window.removeEventListener('mouseup',   onUp);
	}

	// ── widget actions ───────────────────────────────────────────
	function onWidgetMousedown(e, widgetId) {
		if (e.button !== 0) return;
		e.stopPropagation();
		selected = widgetId;
		menuOpen = false;
		startDrag('move', widgetId, e);
	}

	function onResizeMousedown(e, widgetId) {
		if (e.button !== 0) return;
		e.stopPropagation();
		selected = widgetId;
		startDrag('resize', widgetId, e);
	}

	function removeWidget(e, widgetId) {
		e.stopPropagation();
		standbyConfig.widgets = standbyConfig.widgets.filter(w => w.id !== widgetId);
		selected = null;
	}

	function addWidget(type, e) {
		e.stopPropagation();
		const { colStart, rowStart } = findFreeSpot();
		const widget = { id: ++_nextId, type, colStart, rowStart, wSteps: 2, hSteps: 2 };
		if (type === 'clock') widget.showSeconds = false;
		standbyConfig.widgets.push(widget);
		menuOpen = false;
	}

	function deselect() {
		if (_justDragged) { _justDragged = false; return; }
		selected = null;
		menuOpen = false;
	}

	function sp(e) { e.stopPropagation(); }
</script>

<!-- svelte-ignore a11y_click_events_have_key_events a11y_no_static_element_interactions -->
<div
	class="page"
	class:drag-move={dragType === 'move'}
	class:drag-resize={dragType === 'resize'}
	bind:this={pageEl}
	onclick={deselect}
>

	<!-- Dotted grid lines -->
	<svg class="grid" viewBox="0 0 8 8" preserveAspectRatio="none">
		{#each [1,2,3,4,5,6,7] as i}
			<line x1={i} y1="0" x2={i} y2="8" />
			<line x1="0" y1={i} x2="8" y2={i} />
		{/each}
	</svg>

	<!-- Widgets -->
	{#each standbyConfig.widgets as w (w.id)}
		<div
			class="widget-wrap"
			style="
				left:   {w.colStart * 12.5}%;
				top:    {w.rowStart * 12.5}%;
				width:  {w.wSteps  * 12.5}%;
				height: {w.hSteps  * 12.5}%;
			"
		>
			<!-- svelte-ignore a11y_no_static_element_interactions -->
			<div
				class="widget"
				class:selected={selected === w.id}
				style="background: {w.color ?? DEFAULT_COLOR}"
				onmousedown={(e) => onWidgetMousedown(e, w.id)}
				onclick={sp}
			>
				<WidgetContent widget={w} editable />
			</div>

			<!-- SE resize handle -->
			<div class="resize-handle" onmousedown={(e) => onResizeMousedown(e, w.id)}></div>

			<!-- Selection controls -->
			{#if selected === w.id}
				<!-- svelte-ignore a11y_no_static_element_interactions -->
				<div class="controls" onclick={sp}>
					<button class="x-btn" onclick={(e) => removeWidget(e, w.id)}>✕</button>
					{#if w.type === 'clock'}
						<label class="sec-label">
							<input type="checkbox" bind:checked={w.showSeconds} />
							sec
						</label>
					{/if}
					<div class="palette">
						{#each PALETTE as c}
							<button
								class="swatch"
								class:active={( w.color ?? DEFAULT_COLOR) === c}
								style="background: {c}"
								onclick={(e) => { sp(e); w.color = c; }}
							></button>
						{/each}
					</div>
				</div>
			{/if}
		</div>
	{/each}

	<!-- Hamburger menu -->
	<!-- svelte-ignore a11y_no_static_element_interactions -->
	<div class="menu-wrap" onclick={sp}>
		<button class="hamburger" onclick={(e) => { sp(e); menuOpen = !menuOpen; }}>
			<span></span><span></span><span></span>
		</button>
		{#if menuOpen}
			<div class="menu">
				{#each WIDGET_TYPES as { type, label }}
					<button class="menu-item" onclick={(e) => addWidget(type, e)}>{label}</button>
				{/each}
			</div>
		{/if}
	</div>

</div>

<style>
	.page {
		position: relative;
		width: 100%;
		height: 100%;
	}

	.page.drag-move   { cursor: grabbing;  user-select: none; }
	.page.drag-resize { cursor: se-resize; user-select: none; }
	.page.drag-move   * { cursor: grabbing  !important; }
	.page.drag-resize * { cursor: se-resize !important; }

	/* ── Grid ─────────────────────────────── */
	.grid {
		position: absolute;
		inset: 0;
		width: 100%;
		height: 100%;
		pointer-events: none;
	}

	.grid line {
		stroke: #bbb;
		stroke-width: 0.02;
		stroke-dasharray: 0.001 0.18;
		stroke-linecap: round;
	}

	/* ── Widget ───────────────────────────── */
	.widget-wrap {
		position: absolute;
		overflow: visible;
	}

	.widget {
		position: absolute;
		inset: 0;
		color: #fff;
		border-radius: 12px;
		display: flex;
		align-items: center;
		justify-content: center;
		container-type: size;
		overflow: hidden;
		cursor: grab;
		border: 1px solid rgba(255, 255, 255, 0.06);
		box-shadow: 0 6px 20px rgba(0, 0, 0, 0.35), 0 2px 6px rgba(0, 0, 0, 0.2);
		color: #fff;
	}

	.widget.selected {
		outline: 2px solid #0077ff;
		outline-offset: 2px;
	}

	/* ── Resize handle ────────────────────── */
	.resize-handle {
		position: absolute;
		bottom: 0;
		right: 0;
		width: 14px;
		height: 14px;
		cursor: se-resize;
		background: linear-gradient(135deg, transparent 50%, rgba(255,255,255,0.4) 50%);
		border-radius: 0 0 6px 0;
		z-index: 2;
	}

	/* ── Controls ─────────────────────────── */
	.controls {
		position: absolute;
		top: 4px;
		right: 4px;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 4px;
		z-index: 3;
	}

	.x-btn {
		width: 22px;
		height: 22px;
		border-radius: 50%;
		background: #dd0000;
		color: #fff;
		border: none;
		cursor: pointer;
		font-size: 13px;
		line-height: 1;
		padding: 0;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.sec-label {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 1px;
		font-size: 9px;
		color: #fff;
		cursor: pointer;
		user-select: none;
	}

	.sec-label input { margin: 0; width: 12px; height: 12px; cursor: pointer; }

	/* ── Color palette ────────────────────── */
	.palette {
		display: grid;
		grid-template-columns: repeat(4, 1fr);
		gap: 3px;
		margin-top: 2px;
	}

	.swatch {
		width: 14px;
		height: 14px;
		border-radius: 50%;
		border: 2px solid transparent;
		cursor: pointer;
		padding: 0;
	}

	.swatch.active {
		border-color: #fff;
		box-shadow: 0 0 0 1px #333;
	}

	/* ── Hamburger ────────────────────────── */
	.menu-wrap {
		position: absolute;
		top: 8px;
		right: 8px;
		z-index: 20;
	}

	.hamburger {
		width: 28px;
		height: 28px;
		background: rgba(255,255,255,0.92);
		border: 1px solid #ccc;
		border-radius: 4px;
		cursor: pointer;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		gap: 3px;
		padding: 0;
	}

	.hamburger span {
		display: block;
		width: 14px;
		height: 1.5px;
		background: #333;
		border-radius: 1px;
	}

	.menu {
		position: absolute;
		top: 32px;
		right: 0;
		background: #fff;
		border: 1px solid #ccc;
		border-radius: 4px;
		padding: 4px;
		min-width: 110px;
		box-shadow: 0 2px 8px rgba(0,0,0,0.12);
	}

	.menu-item {
		display: block;
		width: 100%;
		text-align: left;
		padding: 5px 8px;
		background: none;
		border: none;
		cursor: pointer;
		font-size: 12px;
		border-radius: 3px;
	}

	.menu-item:hover { background: #f0f0f0; }
</style>
