<script>
	/**
	
	 * @type {boolean}
	 */
	export let isPlaying;

	/**
	 * @type {() => any}
	 */
	export let moveToNextTrack;

	/**
	 * @type {any}
	 */
	export let togglePlayback;
	/**
	 * @type {(arg0: any) => any}
	 */
	export let loadTrack;
	/**
	 * @type {number}
	 */
	export let currentTrackIndex;
	/**
	 * @type {any}
	 */
	export let shufflePlaylist;
	/**
	 * @type {any}
	 */
	export let toggleRightDrawer;
	/**
	 * @type {any}
	 */
	export let toggleLeftDrawer;
	/**
	 * @type {any}
	 */
	export let toggleBottomDrawer;
</script>

<div
	class="bg-gradient-to-t from-black to-green-600/80 text-black rounded-b-xl grid grid-cols-7 items-center border-2 border-zinc-400/20"
>

	<button type="button" class="mx-auto" on:click={toggleLeftDrawer}>
		<img
			src="/svg/eq.svg"
			class="w-8 h-8 transition duration-300 ease-in-out transform hover:scale-125 shadow-lg"
			alt="eq"
		/>
	</button>
	<button type="button" class="mx-auto" on:click={toggleBottomDrawer}>
		<img
			src="/svg/showscreen.svg"
			class="w-8 h-8 transition duration-300 ease-in-out transform hover:scale-125 shadow-lg"
			alt="show"
		/>
	</button>

	<button type="button" class="mx-auto" on:click={() => loadTrack(currentTrackIndex - 1)}>
		<img
			src="/svg/skiplast.svg"
			class="w-10 h-10 transition duration-300 ease-in-out transform hover:scale-125"
			alt="last"
		/>
	</button>
	<button on:click={togglePlayback} class="play-pause-button">
		{#if isPlaying}
			<button type="button" class="mx-auto">
				<img
					src="/svg/pause.svg"
					class="w-14 h-14 transition duration-300 ease-in-out transform hover:scale-125"
					alt="pause"
				/>
			</button>
		{:else}
			<button type="button" class="mx-auto">
				<img
					src="/svg/play.svg"
					class="w-14 h-14 transition duration-300 ease-in-out transform hover:scale-125"
					alt="play"
				/>
			</button>
		{/if}
	</button>
	<button type="button" class="mx-auto" on:click={moveToNextTrack()}>
		<img
			src="/svg/skipnext.svg"
			class="w-10 h-10 transition duration-300 ease-in-out transform hover:scale-125"
			alt="next"
		/>
	</button>
	<button type="button" class="mx-auto" on:click={shufflePlaylist}>
		<img
			src="/svg/shuffle.svg"
			class="w-8 h-8 transition duration-300 ease-in-out transform hover:scale-125"
			alt="shuffle"
		/>
	</button>

	<button type="button" class="mx-auto" on:click={toggleRightDrawer}>
		<img
			src="/svg/playlist.svg"
			class="w-8 h-8 transition duration-300 ease-in-out transform hover:scale-125"
			alt="playlist"
		/>
	</button>
</div>
