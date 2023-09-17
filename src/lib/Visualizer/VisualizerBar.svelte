<script>
	/**
	 * @type {any}
	 */
	export let numBars;
	/**
	 * @type {any}
	 */
	export let isPlaying;
</script>

<div class="equalizer w-full rotate-90 transform-gpu h-96 pb-2 pt-2">
	{#each Array.from({ length: numBars }) as _, index}
		<div
			class="equalizer-bar border border-white/20 rounded-sm ring ring-black/10 shadow-lg shadow-white/40"
			id={'bar-' + index}
			class:gradient-bar-playing={isPlaying}
			style="animation-delay: {index * 0.05}s; transform-origin: center bottom;"
		/>
	{/each}
</div>

<style>
	.equalizer {
		display: flex;
		justify-content: center;
		align-items: center;
	}

	.equalizer-bar {
		width: 10px;
		background-color: transparent;
		margin: 0 0px;
		animation: equalizer-pulse 0.2s alternate infinite, gradient-animation 2s infinite;
	}

	.gradient-bar-playing {
		background: radial-gradient(#000000, #034b18);
		filter: blur(1px);
	}
</style>
