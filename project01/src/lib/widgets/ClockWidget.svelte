<script>
	let { widget } = $props();

	let now = $state(new Date());
	$effect(() => {
		const id = setInterval(() => { now = new Date(); }, 1000);
		return () => clearInterval(id);
	});
</script>

<span class="clock" style="font-size: min(55cqb, {widget.showSeconds ? 13 : 18}cqi)">
	{now.toLocaleTimeString([], widget.showSeconds
		? { hour: '2-digit', minute: '2-digit', second: '2-digit' }
		: { hour: '2-digit', minute: '2-digit' })}
</span>

<style>
	.clock {
		font-family: monospace;
		font-weight: bold;
		white-space: nowrap;
		color: inherit;
	}
</style>
