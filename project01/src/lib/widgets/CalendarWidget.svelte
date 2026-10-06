<script>
	let { widget } = $props();

	let now = $state(new Date());
	$effect(() => {
		const id = setInterval(() => { now = new Date(); }, 60_000);
		return () => clearInterval(id);
	});

	const MONTHS    = ['January','February','March','April','May','June','July','August','September','October','November','December'];
	const DAY_HDRS  = ['Su','Mo','Tu','We','Th','Fr','Sa'];

	let year        = $derived(now.getFullYear());
	let month       = $derived(now.getMonth());
	let today       = $derived(now.getDate());
	let firstDay    = $derived(new Date(year, month, 1).getDay());
	let daysInMonth = $derived(new Date(year, month + 1, 0).getDate());
	let cells       = $derived(
		Array.from({ length: 42 }, (_, i) => {
			const d = i - firstDay + 1;
			return d >= 1 && d <= daysInMonth ? d : null;
		})
	);
</script>

<div class="cal">
	<div class="header" style="font-size: min(10cqb, 8cqi)">{MONTHS[month]} {year}</div>
	<div class="grid">
		{#each DAY_HDRS as h}
			<div class="day-hdr" style="font-size: min(7cqb, 6cqi)">{h}</div>
		{/each}
		{#each cells as cell}
			<div class="cell" class:today={cell === today} class:empty={cell === null} style="font-size: min(9cqb, 7cqi)">
				{cell ?? ''}
			</div>
		{/each}
	</div>
</div>

<style>
	.cal {
		display: flex;
		flex-direction: column;
		width: 100%;
		height: 100%;
		padding: 4px 6px;
		box-sizing: border-box;
		color: inherit;
	}

	.header {
		text-align: center;
		font-family: sans-serif;
		font-weight: bold;
		padding-bottom: 3px;
		border-bottom: 1px solid rgba(255,255,255,0.15);
		flex-shrink: 0;
	}

	.grid {
		flex: 1;
		display: grid;
		grid-template-columns: repeat(7, 1fr);
		align-content: space-around;
	}

	.day-hdr {
		text-align: center;
		font-family: sans-serif;
		opacity: 0.5;
		padding: 1px 0;
	}

	.cell {
		text-align: center;
		font-family: monospace;
		display: flex;
		align-items: center;
		justify-content: center;
		border-radius: 50%;
		aspect-ratio: 1;
		margin: auto;
		width: min(11cqb, 9cqi);
		height: min(11cqb, 9cqi);
	}

	.cell.today {
		background: #0077ff;
		font-weight: bold;
	}

	.cell.empty {
		visibility: hidden;
	}
</style>
