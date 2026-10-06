<script>
	let { widget } = $props();

	const WMO = {
		0:  [' ', 'Clear'],          1:  ['🌤', 'Mostly Clear'],   2:  [' ', 'Partly Cloudy'],
		3:  [' ', 'Overcast'],        45: [' ', 'Fog'],            48: [' ', 'Icy Fog'],
		51: ['🌦', 'Light Drizzle'],   53: ['🌦', 'Drizzle'],        55: ['🌧', 'Heavy Drizzle'],
		61: ['🌧', 'Light Rain'],      63: ['🌧', 'Rain'],           65: ['🌧', 'Heavy Rain'],
		71: ['🌨', 'Light Snow'],      73: ['🌨', 'Snow'],           75: [' ', 'Heavy Snow'],
		80: ['🌦', 'Showers'],         81: ['🌧', 'Heavy Showers'],  82: ['⛈', 'Violent Showers'],
		95: ['⛈', 'Thunderstorm'],    96: ['⛈', 'Hail'],           99: ['⛈', 'Heavy Hail'],
	};

	let weather = $state(null);

	async function load(lat, lon) {
		const url = `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&current=temperature_2m,apparent_temperature,weather_code,wind_speed_10m&temperature_unit=fahrenheit&wind_speed_unit=mph`;
		const json = await fetch(url).then(r => r.json());
		weather = json.current;
	}

	$effect(() => {
		const tryLoad = () => navigator.geolocation?.getCurrentPosition(
			p => load(p.coords.latitude, p.coords.longitude),
			()  => load(40.71, -74.01)
		);
		tryLoad();
		const id = setInterval(tryLoad, 10 * 60 * 1000);
		return () => clearInterval(id);
	});
</script>

{#if weather}
	{@const [icon, cond] = WMO[weather.weather_code] ?? [' ', 'Unknown']}
	<div class="weather">
		<span class="icon"  style="font-size: min(40cqb, 20cqi)">{icon}</span>
		<span class="temp"  style="font-size: min(28cqb, 16cqi)">{Math.round(weather.temperature_2m)}°F</span>
		<span class="cond"  style="font-size: min(12cqb,  8cqi)">{cond}</span>
		<span class="feels" style="font-size: min(10cqb,  7cqi)">Feels {Math.round(weather.apparent_temperature)}°F · {Math.round(weather.wind_speed_10m)} mph</span>
	</div>
{:else}
	<span class="loading" style="font-size: min(12cqb, 20cqi)">Loading…</span>
{/if}

<style>
	.weather {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		gap: 2px;
		color: inherit;
		text-align: center;
		padding: 4px;
	}

	.icon { line-height: 1; }
	.temp { font-weight: bold; font-family: monospace; color: inherit; }
	.cond, .feels { opacity: 0.8; font-family: sans-serif; color: inherit; }

	.loading {
		font-family: monospace;
		font-weight: bold;
		color: inherit;
	}
</style>
