<script>
	let { widget } = $props();

	let now = $state(new Date());
	$effect(() => {
		const id = setInterval(() => { now = new Date(); }, 60_000);
		return () => clearInterval(id);
	});
</script>

<div class="date">
	<span class="weekday" style="font-size: min(18cqb, 16cqi)">
		{now.toLocaleDateString([], { weekday: 'long' })}
	</span>
	<span class="monthday" style="font-size: min(13cqb, 11cqi)">
		{now.toLocaleDateString([], { month: 'long', day: 'numeric' })}
	</span>
	<span class="year" style="font-size: min(10cqb, 8cqi)">
		{now.getFullYear()}
	</span>
</div>

<style>
	.date {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		gap: 2px;
		color: inherit;
		text-align: center;
	}

	.weekday  { font-family: sans-serif; font-weight: bold; }
	.monthday { font-family: sans-serif; opacity: 0.9; }
	.year     { font-family: monospace; opacity: 0.6; }
</style>
