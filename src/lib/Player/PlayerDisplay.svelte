<script>
	export let playlist;
	/**
	 * @type {string | any[]}
	 */
	export let currentTrackName;
	/**
	 * @type {any}
	 */
	export let currentTime;
	/**
	 * @type {(arg0: any) => void}
	 */
	export let formatTime;
	/**
	 * @type {any}
	 */
	export let duration;
	/**
	 * @type {any}
	 */
	export let seekPosition;
	/**
	 * @type {any}
	 */
	export let seek;
	/**
	 * @type {any}
	 */
	export let volume;
	/**
	 * @type {any}
	 */
	export let changeVolume;
	/**
	 * @type {any}
	 */
	export let balance;
	/**
	 * @type {any}
	 */
	export let changeVolumeBalance;
</script>

<div
	class="bg-gradient-to-b from-neutral-900 to-green-800 rounded-t-xl backdrop-blur-md w-full border-double border-2 border-zinc-400/20 shadow-md shadow-zinc-400/50"
>
<div class="border border-black/40 shadow-md rounded-t-xl">
	<button
		class="absolute top-2 right-2 text-gray-400 hover:text-black hover:bg-gray-200/50 transition rounded-full focus:outline-none"
	>
		<svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" viewBox="0 0 20 20" fill="currentColor">
			<path
				d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z"
			/>
		</svg>
	</button>
	<div class="w-full text-left pt-4 px-4">
		<h2 class="text-2xl font-bold italic text-black/80 animate-pulse">Now Playing:</h2>
		{#if playlist.length > 0}
			<div>
				<p class="text-sm text-gray-400 truncate uppercase font-light">{currentTrackName}</p>
			</div>
		{:else}
			<div>
				<p class="text-sm text-gray-400 truncate uppercase font-light">No Track Playing.</p>
			</div>
		{/if}
	</div>

	<div class="flex flex-wrap items-center">
		<div class="w-1/3">
			<div class="flex items-center space-x-2 scale-75 translate-y-3">
				<span class="text-black font-bold">L</span>
				<input
					type="range"
					min="0"
					max="1"
					step="0.25"
					bind:value={balance}
					on:input={changeVolumeBalance}
					class="custom-range w-24 h-3 shadow-md shadow-black/50"
				/>
				<span class="text-black font-bold">R</span>
			</div>
		</div>
		<div class="w-1/3">
			<div class="text-center text-sm font-extrabold mt-5">
				<span class="text-gray-900">{formatTime(currentTime)} / {formatTime(duration)}</span>
			</div>
		</div>
		<div class="w-1/3">
			<div class="flex items-center space-x-2 scale-75 translate-y-3">
				<span class="text-black text-2xl font-bold">-</span>
				<input
					type="range"
					min="0"
					max="0.9"
					step="0.1"
					bind:value={volume}
					on:input={changeVolume}
					class="custom-range w-24 h-3 shadow-md shadow-black/50"
				/>
				<span class="text-black text-2xl font-bold">+</span>
			</div>
		</div>
	</div>

	<div class="w-full translate-y-2">
		<input
			type="range"
			min="0"
			max="100"
			step="0.1"
			bind:value={seekPosition}
			on:input={seek}
			class="custom-range border-b border-zinc-400/80 shadow-lg shadow-zinc-400/90"
			disabled
		/>
	</div>
</div>
</div>

<style>
	.custom-range {
		appearance: none;
		-webkit-appearance: none;
		-moz-appearance: none;
		width: 100%;
		height: 10px;
		background: #00000086;
		outline: none;
		border-bottom: 1px solid rgba(192, 192, 192, 0.404);
	}

	.custom-range::-webkit-slider-thumb {
		-webkit-appearance: none;
		appearance: none;
		width: 0;
		height: 0;
		border-left: 10px solid transparent;
		border-right: 10px solid transparent;
		border-bottom: 15px solid #111111;
		position: relative;
	}
</style>
