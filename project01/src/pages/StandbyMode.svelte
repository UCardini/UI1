<script>
	import { getContext } from 'svelte';
	import WidgetContent from '../lib/WidgetContent.svelte';

	const standbyConfig = getContext('standbyConfig');

	const DEFAULT_COLOR = '#3a3a3a';
</script>

<div class="standby">
	{#each standbyConfig.widgets as w (w.id)}
		<div class="widget" style="
			left:       {w.colStart * 12.5}%;
			top:        {w.rowStart * 12.5}%;
			width:      {w.wSteps  * 12.5}%;
			height:     {w.hSteps  * 12.5}%;
			background: {w.color ?? DEFAULT_COLOR};
		">
			<WidgetContent widget={w} />
		</div>
	{/each}
</div>

<style>
	.standby {
		position: relative;
		width: 100%;
		height: 100%;
	}

	.widget {
		position: absolute;
		border-radius: 12px;
		display: flex;
		align-items: center;
		justify-content: center;
		container-type: size;
		overflow: hidden;
		border: 1px solid rgba(255, 255, 255, 0.06);
		box-shadow: 0 6px 20px rgba(0, 0, 0, 0.35), 0 2px 6px rgba(0, 0, 0, 0.2);
		color: #fff;
	}
</style>
