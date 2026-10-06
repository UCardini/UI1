<script>
	let { widget, editable = false } = $props();

	function load() {
		try { return JSON.parse(localStorage.getItem('smartfloor_todo')) ?? []; }
		catch { return []; }
	}

	let items = $state(load());
	let input = $state('');

	$effect(() => {
		localStorage.setItem('smartfloor_todo', JSON.stringify(items));
	});

	function add() {
		const text = input.trim();
		if (!text) return;
		items.push({ id: Date.now(), text, done: false });
		input = '';
	}

	function toggle(id) {
		const item = items.find(i => i.id === id);
		if (item) item.done = !item.done;
	}

	function remove(id) {
		items = items.filter(i => i.id !== id);
	}
</script>

<!-- svelte-ignore a11y_no_static_element_interactions -->
<div class="todo">
	<div class="header" style="font-size: min(10cqb, 7cqi)">To-Do</div>

	<!-- svelte-ignore a11y_no_static_element_interactions -->
	<ul class="list" onmousedown={(e) => editable && e.stopPropagation()}>
		{#each items as item (item.id)}
			<li class:done={item.done}>
				{#if editable}
					<button class="check" onclick={() => toggle(item.id)}>
						{item.done ? '✓' : '○'}
					</button>
				{/if}
				<span class="text" style="font-size: min(8cqb, 6cqi)">{item.text}</span>
				{#if editable}
					<button class="del" onclick={() => remove(item.id)}>✕</button>
				{/if}
			</li>
		{:else}
			<li class="empty" style="font-size: min(7cqb, 5.5cqi)">No items</li>
		{/each}
	</ul>

	{#if editable}
	<div class="add-row" onmousedown={(e) => e.stopPropagation()}>
		<input
			class="add-input"
			bind:value={input}
			placeholder="Add item…"
			style="font-size: min(8cqb, 6cqi)"
			onkeydown={(e) => e.key === 'Enter' && add()}
		/>
		<button class="add-btn" onclick={add} style="font-size: min(10cqb, 8cqi)">+</button>
	</div>
	{/if}
</div>

<style>
	.todo {
		display: flex;
		flex-direction: column;
		width: 100%;
		height: 100%;
		color: inherit;
		box-sizing: border-box;
	}

	.header {
		font-family: sans-serif;
		font-weight: bold;
		text-align: center;
		padding: 4px 0 2px;
		border-bottom: 1px solid rgba(255,255,255,0.15);
		flex-shrink: 0;
		cursor: grab;
	}

	.list {
		list-style: none;
		margin: 0;
		padding: 4px 6px;
		flex: 1;
		overflow-y: auto;
		display: flex;
		flex-direction: column;
		gap: 3px;
	}

	.list li {
		display: flex;
		align-items: center;
		gap: 4px;
	}

	.list li.done .text {
		text-decoration: line-through;
		opacity: 0.45;
	}

	.list li.empty {
		opacity: 0.4;
		font-family: sans-serif;
		justify-content: center;
	}

	.text {
		flex: 1;
		font-family: sans-serif;
		line-height: 1.3;
		min-width: 0;
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
	}

	.check, .del {
		background: none;
		border: none;
		color: inherit;
		cursor: pointer;
		padding: 0 2px;
		font-size: min(9cqb, 7cqi);
		flex-shrink: 0;
		line-height: 1;
	}

	.del { opacity: 0.4; }
	.del:hover { opacity: 1; }

	.add-row {
		display: flex;
		border-top: 1px solid rgba(255,255,255,0.12);
		flex-shrink: 0;
	}

	.add-input {
		flex: 1;
		background: rgba(255,255,255,0.08);
		border: none;
		color: inherit;
		padding: 4px 6px;
		font-family: sans-serif;
		outline: none;
		min-width: 0;
	}

	.add-input::placeholder { opacity: 0.4; }

	.add-btn {
		background: rgba(255,255,255,0.1);
		border: none;
		color: inherit;
		padding: 0 8px;
		cursor: pointer;
		font-family: monospace;
		flex-shrink: 0;
	}

	.add-btn:hover { background: rgba(255,255,255,0.15); }
</style>
