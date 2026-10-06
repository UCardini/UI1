<script>
	let { widget } = $props();

	// How many people are expected home at this hour
	function baseline() {
		const h = new Date().getHours();
		if (h >= 9  && h < 17) return 1;  // work hours: mostly out
		if (h >= 22 || h < 6)  return 4;  // night: mostly in
		return 3;                           // morning / evening
	}

	function load() {
		try {
			const v = JSON.parse(localStorage.getItem('smartfloor_occupancy'));
			return typeof v === 'number' ? v : baseline();
		} catch { return baseline(); }
	}

	const MAX = 8;
	let count = $state(load());

	$effect(() => {
		localStorage.setItem('smartfloor_occupancy', JSON.stringify(count));
	});

	// Simulate people entering / leaving
	$effect(() => {
		let id;
		function tick() {
			const base  = baseline();
			// Bias the random walk toward the time-of-day baseline
			const delta = count < base ? 1 : count > base ? -1 : (Math.random() < 0.5 ? 1 : -1);
			count = Math.max(0, Math.min(MAX, count + delta));
			// Next tick: 1 – 4 minutes
			id = setTimeout(tick, (1 + Math.random() * 3) * 60_000);
		}
		id = setTimeout(tick, (1 + Math.random() * 3) * 60_000);
		return () => clearTimeout(id);
	});
</script>

<div class="occ">
	<span class="num"  style="font-size: min(52cqb, 40cqi)">{count}</span>
	<span class="label" style="font-size: min(12cqb, 9cqi)">people home</span>
</div>

<style>
	.occ {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		width: 100%;
		height: 100%;
		color: inherit;
		gap: 2px;
	}

	.num {
		font-family: monospace;
		font-weight: bold;
		line-height: 1;
	}

	.label {
		font-family: sans-serif;
		opacity: 0.65;
		letter-spacing: 0.05em;
	}
</style>
