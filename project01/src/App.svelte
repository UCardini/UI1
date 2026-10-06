<script>
	import { setContext } from 'svelte';
	import Home from './pages/Home.svelte';
	import StandbyMode from './pages/StandbyMode.svelte';
	import InteractiveMode1 from './pages/InteractiveMode1.svelte';
	import InteractiveMode2 from './pages/InteractiveMode2.svelte';

	const pages = {
		home: Home,
		standby: StandbyMode,
		interactive1: InteractiveMode1,
		interactive2: InteractiveMode2,
	};

	let current = $state('standby');

	// ── User profiles ────────────────────────────────────────────
	const PROFILES = [
		{ id: 'austin',  name: 'Austin',  startWeight: 215.3, currentWeight: 189.0 },
		{ id: 'sarah',   name: 'Sarah',   startWeight: 147.2, currentWeight: 145.8 },
		{ id: 'marcus',  name: 'Marcus',  startWeight: 175.0, currentWeight: 192.4 },
		{ id: 'emma',    name: 'Emma',    startWeight: 168.3, currentWeight: 154.7 },
	];
	const profileState = $state({ active: PROFILES[0], list: PROFILES });
	setContext('profileState', profileState);

	const LS_KEY = 'smartfloor_widgets';

	function loadWidgets() {
		try { return JSON.parse(localStorage.getItem(LS_KEY)) ?? []; }
		catch { return []; }
	}

	const standbyConfig = $state({ widgets: loadWidgets() });
	setContext('standbyConfig', standbyConfig);

	$effect(() => {
		localStorage.setItem(LS_KEY, JSON.stringify(standbyConfig.widgets));
	});

</script>

<div class="stage">
	<div class="device">
		<!-- <h1>SMART FLOOR</h1> -->

		<nav>
			<button onclick={() => current = 'home'}>Home</button>
			<button onclick={() => current = 'standby'}>Standby</button>
			<button onclick={() => current = 'interactive1'}>Interactive 1</button>
			<button onclick={() => current = 'interactive2'}>Interactive 2</button>
		</nav>

		<div class="screen">
			<svelte:component this={pages[current]} />
		</div>
	</div>
</div>

<style>
	:global(body) {
		margin: 0;
		background: #1a1a1a;
		display: flex;
		justify-content: center;
		align-items: center;
		min-height: 100vh;
	}

	.stage {
		display: flex;
		justify-content: center;
		align-items: center;
		padding: 2rem;
	}

	.device {
		width: 780px;
		height: 820px;
		background: #fff;
		display: flex;
		flex-direction: column;
		overflow: hidden;
		font-family: sans-serif;
	}

	nav {
		display: flex;
		border-bottom: 1px solid #111;
	}

	nav button {
		flex: 1;
		border-right: 1px solid #111;
		font-size: 10px;
		cursor: pointer;
	}

	nav button:last-child {
		border-right: none;
	}

	nav button:hover {
		background: #111;
		color: white;
	}

	.screen {
		flex: 1;
		overflow: hidden;
		padding: 0;
	}
</style>
