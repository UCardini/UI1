<script>
	let { widget } = $props();

	let news = $state([]);

	function parseXml(text) {
		const doc   = new DOMParser().parseFromString(text, 'text/xml');
		const items = [...doc.querySelectorAll('item, entry')].slice(0, 3);
		news = items.map(el => {
			let image = null;
			for (const child of el.querySelectorAll('*')) {
				if ((child.localName === 'content' || child.localName === 'thumbnail') && child.getAttribute('url')) {
					image = child.getAttribute('url'); break;
				}
				if (child.localName === 'enclosure' && child.getAttribute('type')?.startsWith('image')) {
					image = child.getAttribute('url'); break;
				}
			}
			const pubRaw = el.querySelector('pubDate, published, updated')?.textContent ?? '';
			const pub    = pubRaw ? new Date(pubRaw) : null;
			return { title: el.querySelector('title')?.textContent ?? '', image, pub };
		});
	}

	async function load() {
		const feed = 'https://moxie.foxweather.com/google-publisher/latest.xml';
		try {
			const text = await fetch(feed).then(r => r.text());
			parseXml(text);
		} catch {
			try {
				const text = await fetch(`https://corsproxy.io/?${encodeURIComponent(feed)}`).then(r => r.text());
				parseXml(text);
			} catch(e) {
				console.error('[NewsWidget] fetch failed:', e);
			}
		}
	}

	$effect(() => {
		load();
		const id = setInterval(load, 5 * 60 * 1000);
		return () => clearInterval(id);
	});
</script>

{#if news.length}
	<div class="wrap">
		<div class="header">News</div>
		<ul class="list">
			{#each news as item}
				<li>
					<div class="text">
						<span class="title">{item.title}</span>
						{#if item.pub}
							<span class="date">
								{item.pub.toLocaleDateString([], { month: 'short', day: 'numeric' })} · {item.pub.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })}
							</span>
						{/if}
					</div>
					{#if item.image}
						<img src={item.image} alt="" class="thumb" />
					{/if}
				</li>
			{/each}
		</ul>
	</div>
{:else}
	<span class="loading" style="font-size: min(12cqb, 20cqi)">Loading…</span>
{/if}

<style>
	.wrap {
		display: flex;
		flex-direction: column;
		width: 100%;
		height: 100%;
	}

	.header {
		font-family: sans-serif;
		font-weight: bold;
		font-size: min(10cqb, 7cqi);
		color: inherit;
		text-align: center;
		padding: 4px 0 2px;
		border-bottom: 1px solid rgba(255,255,255,0.15);
		flex-shrink: 0;
	}

	.list {
		list-style: none;
		margin: 0;
		padding: 4px 8px;
		flex: 1;
		display: flex;
		flex-direction: column;
		justify-content: space-around;
		box-sizing: border-box;
		gap: 4px;
	}

	.list li {
		display: flex;
		align-items: center;
		gap: 6px;
		border-left: 2px solid rgba(255,255,255,0.25);
		padding-left: 6px;
	}

	.text {
		flex: 1;
		display: flex;
		flex-direction: column;
		gap: 2px;
		min-width: 0;
	}

	.title {
		font-size: min(5cqb, 4cqi);
		color: inherit;
		font-family: sans-serif;
		line-height: 1.3;
		text-align: left;
	}

	.date {
		font-size: min(5cqb, 4cqi);
		color: rgba(255,255,255,0.45);
		font-family: sans-serif;
	}

	.thumb {
		width: min(18cqb, 14cqi);
		height: min(18cqb, 14cqi);
		object-fit: cover;
		border-radius: 3px;
		flex-shrink: 0;
	}

	.loading {
		font-family: monospace;
		font-weight: bold;
		color: inherit;
	}
</style>
