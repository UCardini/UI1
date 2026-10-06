<script>
	import ClockWidget        from './widgets/ClockWidget.svelte';
	import AnalogClockWidget  from './widgets/AnalogClockWidget.svelte';
	import WeatherWidget  from './widgets/WeatherWidget.svelte';
	import NewsWidget     from './widgets/NewsWidget.svelte';
	import TodoWidget     from './widgets/TodoWidget.svelte';
	import DateWidget     from './widgets/DateWidget.svelte';
	import CalendarWidget from './widgets/CalendarWidget.svelte';
	import OccupancyWidget from './widgets/OccupancyWidget.svelte';

	let { widget, editable = false } = $props();
</script>

{#if widget.type === 'clock2'}
	<AnalogClockWidget {widget} />
{:else if widget.type === 'clock'}
	<ClockWidget {widget} />
{:else if widget.type === 'weather'}
	<WeatherWidget {widget} />
{:else if widget.type === 'news'}
	<NewsWidget {widget} />
{:else if widget.type === 'todo'}
	<TodoWidget {widget} {editable} />
{:else if widget.type === 'date'}
	<DateWidget {widget} />
{:else if widget.type === 'calendar'}
	<CalendarWidget {widget} />
{:else if widget.type === 'occupancy'}
	<OccupancyWidget {widget} />
{/if}
