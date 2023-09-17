<script>
	import { filterValueStore, visualizerStore } from "$lib/store";
	import { blur } from "svelte/transition";
	import { quintInOut } from "svelte/easing";

	/**
	 * @type {any}
	 */
	export let toggleModal;

	/**
	 * @type {any}
	 */
	export let toggleLoopMode;
	/**
	 * @type {any}
	 */
	export let loopModeLabel;

	/**
	 * @type {any}
	 */
	let selectedVisualizer;
	

	function updateVisualizerValue() {
		switch (selectedVisualizer) {
			case "Bar":
				visualizerStore.set("Bar");
				break;
			case "Waveform":
				visualizerStore.set("Waveform");
				break;
			case "DMT":
				visualizerStore.set("DMT");
				break;
		}
	}

	/**
	 * @type {any}
	 */
	let selectedHueRotation;
	

	function updateFilterValue() {
		switch (selectedHueRotation) {
			case "Weed":
				filterValueStore.set("hue-rotate-0");
				break;
			case "Icy":
				filterValueStore.set("hue-rotate-90");
				break;
			case "Barbie":
				filterValueStore.set("hue-rotate-180");
				break;
		}
	}

	
</script>

<div
	transition:blur={{ duration: 400, delay: 200, easing: quintInOut }}
	class="playlist-container max-h-96 border-double border-2 border-zinc-400/20 mt-4 rounded-md bg-gradient-to-t from-neutral-800 to-green-700 backdrop-blur-sm shadow-md shadow-zinc-400/20 overflow-x-hidden"
>
<div class="border border-black/40 rounded-md shadow-md">
	<!-- Playlist Header -->
	<div
		class="playlist-header flex items-center justify-evenly bg-black/50 backdrop-blur-sm border-b border-zinc-400/50 shadow-md shadow-zinc-500/50 text-white p-2 sticky top-0 z-10"
	>
		<button
			class="bg-black/50 ring-1 ring-white/10 ring-inset hover:ring-0 hover:cursor-pointer hover:bg-black/90 border border-white/10 text-xs text-gray-500 font-bold py-1 px-4 rounded-full transition duration-300 ease-in-out shadow-sm shadow-black/50"
		>
			About
		</button>
		<button
			on:click={toggleModal}
			class="bg-black/50 ring-1 ring-white/10 ring-inset hover:ring-0 hover:bg-black/90 border border-white/10 text-xs text-gray-500 font-bold py-1 px-4 rounded-full transition duration-300 ease-in-out shadow-sm shadow-black/50"
		>
			Macros
		</button>
		<h3 class="text-lg text-gray-400 font-semibold">Tuning</h3>
	</div>
	<div class="p-2">
		<h3>Select Skin:</h3>
		<select
			bind:value={selectedHueRotation}
			on:change={updateFilterValue}
			class="bg-black text-white/50"
		>
			<option value="Weed">WeedJoint</option>
			<option value="Icy">Icy_Tower</option>
			<option value="Barbie">BarbieDoll</option>
		</select>
		<h3>Loop Mode:</h3>
		<button class="text-white bg-black" on:click={toggleLoopMode}
			>{loopModeLabel}</button
		>
		<h3>Visualizer:</h3>
		<select
			bind:value={selectedVisualizer}
			on:change={updateVisualizerValue}
			class="bg-black text-white/50"
		>
			<option value="Bar">Mirror Bar</option>
			<option value="Waveform">Waveform</option>
			<option value="DMT">DMT</option>
		</select>
	</div>
</div>
</div>
