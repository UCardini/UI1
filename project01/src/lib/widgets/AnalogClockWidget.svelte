<script>
	let { widget } = $props();

	let now = $state(new Date());
	$effect(() => {
		const id = setInterval(() => { now = new Date(); }, 1000);
		return () => clearInterval(id);
	});

	function hand(pct, len) {
		const a = pct * 2 * Math.PI;
		return { x: Math.sin(a) * len, y: -Math.cos(a) * len };
	}

	let h = $derived(hand((now.getHours() % 12 + now.getMinutes() / 60) / 12, 0.5));
	let m = $derived(hand((now.getMinutes() + now.getSeconds() / 60) / 60, 0.7));
	let s = $derived(hand(now.getSeconds() / 60, 0.8));
</script>

<svg viewBox="-1 -1 2 2" style="width:100%;height:100%;display:block">
	<circle r="0.92" fill="none" stroke="currentColor" stroke-width="0.04" opacity="0.3" />
	{#each [0,1,2,3,4,5,6,7,8,9,10,11] as i}
		{@const a = i / 12 * 2 * Math.PI}
		<line
			x1={Math.sin(a) * 0.78} y1={-Math.cos(a) * 0.78}
			x2={Math.sin(a) * 0.88} y2={-Math.cos(a) * 0.88}
			stroke="currentColor" stroke-width={i % 3 === 0 ? 0.05 : 0.02} opacity="0.4"
		/>
	{/each}
	<line x1="0" y1="0" x2={h.x} y2={h.y} stroke="currentColor" stroke-width="0.09" stroke-linecap="round" />
	<line x1="0" y1="0" x2={m.x} y2={m.y} stroke="currentColor" stroke-width="0.06" stroke-linecap="round" />
	<line x1="0" y1="0" x2={s.x} y2={s.y} stroke="#e74c3c"     stroke-width="0.03" stroke-linecap="round" />
	<circle r="0.06" fill="currentColor" />
</svg>
