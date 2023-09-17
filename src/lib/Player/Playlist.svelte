<script>
	import { fade, blur } from 'svelte/transition';
	import { quintInOut } from 'svelte/easing';

	/**
	 * @type {HTMLUListElement}
	 */
	let playlistContainer;

	/**
	 * @type {string | any[]}
	 */
	export let playlist;
	/**
	 * @type {(arg0: number) => any}
	 */
	export let loadTrack;
	/**
	 * @type {() => any}
	 */
	export let clearPlaylist;
	/**
	 * @type {number}
	 */
	export let currentTrackIndex;
	/**
	 * @type {(arg0: any) => void}
	 */
	export let removeSong;

	/**
	 * @param {number} index
	 */
	function handleRemove(index) {
		removeSong(index);
	}

	$: {
		if (playlistContainer && currentTrackIndex !== -1) {
			const currentTrackElement = playlistContainer.querySelector(
				`.playlist-item:nth-child(${currentTrackIndex + 1})`
			);
			if (currentTrackElement) {
				currentTrackElement.scrollIntoView({
					behavior: 'smooth',
					block: 'center',
					inline: 'start'
				});
			}
		}
	}
	function checkOverflow() {
    const container = document.querySelector('.track-name-container');
    const trackName = document.querySelector('.track-name');

    if (container && trackName) {
        const isOverflowing = trackName.scrollWidth > container.clientWidth;

        if (isOverflowing) {
            trackName.classList.add('scrolling');
        } else {
            trackName.classList.remove('scrolling');
        }
    }
}




</script>

<div
	transition:blur={{ duration: 400, delay: 200, easing: quintInOut }}
	class="playlist-container transition-all duration-300 max-h-96 border-double border-2 border-zinc-400/20 mt-4 rounded-md bg-gradient-to-t from-neutral-800 to-green-700 shadow-md shadow-zinc-400/20 overflow-x-hidden"
>
<div class="border-2 border-black/40 rounded-md shadow-md">
	<!-- Playlist Header -->
	<div
		class="playlist-header flex items-center justify-evenly bg-black/50 backdrop-blur-sm border-b border-zinc-400/50 shadow-md shadow-zinc-500/50 text-white p-2 sticky top-0 z-10"
	>
		<h3 class="text-lg text-gray-400 font-semibold">Playlist</h3>

		<!-- svelte-ignore a11y-click-events-have-key-events -->
		<!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
		<label
			class="bg-black/50 ring-1 ring-white/10 ring-inset hover:ring-0 hover:cursor-pointer hover:bg-black/90 border border-white/10 text-xs text-gray-500 font-bold py-1 px-4 rounded-full transition duration-300 ease-in-out shadow-sm shadow-black/50"
			for="thefile"
		>
			Load
		</label>

		<button
			on:click={clearPlaylist}
			class="bg-black/50 ring-1 ring-white/10 ring-inset hover:ring-0 hover:bg-black/90 border border-white/10 text-xs text-gray-500 font-bold py-1 px-4 rounded-full transition duration-300 ease-in-out shadow-sm shadow-black/50"
		>
			Clear
		</button>
	</div>

	{#if playlist.length > 0}
		<ul class="playlist-items divide-y divide-gray-500" bind:this={playlistContainer}>
			{#each playlist as track, index (track.url)}
				<!-- svelte-ignore a11y-click-events-have-key-events -->
				<!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
				<li
					transition:fade={{ duration: (index % 7) * 100, easing: quintInOut }}
					class="playlist-item p-1 hover:bg-black/20 cursor-pointer transition duration-300"
				>
					<div class="flex items-center space-x-1">
						<img
							on:click={() => handleRemove(index)}
							src="/svg/delete.svg"
							class="w-4 h-4 hover:animate-ping transition-all duration-200 opacity-50 hover:opacity-100"
							alt="remove song"
						/>

						<!-- svelte-ignore a11y-no-static-element-interactions -->
						<div
							class="flex-1 w-60 border-l-2 border-gray-500/30"
							on:click={() => loadTrack(index)}
							class:playing={index === currentTrackIndex}
						>
							<p
								class="text-sm font-bold ml-1  uppercase transition-colors duration-200 truncate {index ===
								currentTrackIndex
									? 'text-gray-300'
									: 'text-black/60'}"
							>
								{track.name}
							</p>
							<p
								class="text-xs font-light ml-1 italic  transition-colors duration-200 {index ===
								currentTrackIndex
									? 'text-gray-400'
									: 'text-black/70'}"
							>
								Track: {index + 1} / {playlist.length} ({track.duration}) // {track.kbps}
								{track.format}
							</p>
						</div>
					</div>
				</li>
			{/each}
		</ul>
	{/if}
</div>
</div>

<style>
	.playlist-container::-webkit-scrollbar {
		width: 0.5em;
	}

	.playlist-container::-webkit-scrollbar-track {
		background-color: rgba(0, 0, 0, 0.671);
	}

	.playlist-container::-webkit-scrollbar-thumb {
		background-color: #666666f5;
		border-radius: 0.25em;
	}
</style>
