<script>
	import { fade, scale } from "svelte/transition";
	import { cubicInOut, quintInOut } from "svelte/easing";
	export let isOpen = false;
	/**
	 * @type {any}
	 */
	export let toggleNotifications;
	/**
	 * @type {any}
	 */
	export let notificationsEnabled;
</script>

<div
	transition:fade={{ duration: 200, delay: 200, easing: cubicInOut }}
	class="fixed inset-0 flex items-center justify-center z-50 bg-black/50"
	class:hidden={!isOpen}
>
	<div
		transition:scale={{
			duration: 500,
			delay: 100,
			opacity: 0,
			start: 0.5,
			easing: quintInOut,
		}}
		class="w-1/3 select-none bg-gradient-to-b mx-auto text-center from-neutral-900/90 to-green-800/90 rounded-t-xl rounded-b-md border-2 border-zinc-400/20 shadow-md shadow-zinc-400/50 relative"
	>
		<button
			class="absolute top-2 right-2 text-gray-400 hover:text-gray-600 hover:bg-gray-200 transition rounded-full focus:outline-none"
			on:click={() => (isOpen = false)}
		>
			<svg
				xmlns="http://www.w3.org/2000/svg"
				class="h-4 w-4"
				viewBox="0 0 20 20"
				fill="currentColor"
			>
				<path
					d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z"
				/>
			</svg>
		</button>

		<div class="overflow-x-auto">
			<table class="min-w-full divide-y-2 divide-gray-400 text-sm">
				<thead>
					<tr>
						<th
							class="whitespace-nowrap px-4 py-2 font-medium text-gray-200"
						>
							Hotkey
						</th>
						<th
							class="whitespace-nowrap px-4 py-2 font-medium text-gray-200"
						>
							Function
						</th>
					</tr>
				</thead>

				<tbody class="divide-y divide-gray-400">
					<tr>
						<td
							class="whitespace-nowrap px-4 py-2 font-medium text-gray-900"
						>
							<kbd>Spacebar</kbd>
						</td>
						<td class="whitespace-nowrap px-4 py-2 text-gray-400"
							>Play/Pause</td
						>
					</tr>

					<tr>
						<td
							class="whitespace-nowrap px-4 py-2 font-medium text-gray-900"
						>
							<kbd>Shift</kbd>
						</td>
						<td class="whitespace-nowrap px-4 py-2 text-gray-400"
							>Visualizer</td
						>
					</tr>

					<tr>
						<td
							class="whitespace-nowrap px-4 py-2 font-medium text-gray-900"
						>
							<kbd>Arrow Up</kbd>
						</td>
						<td class="whitespace-nowrap px-4 py-2 text-gray-400"
							>Volume Up</td
						>
					</tr>

					<tr>
						<td
							class="whitespace-nowrap px-4 py-2 font-medium text-gray-900"
						>
							<kbd>Arrow Down</kbd>
						</td>
						<td class="whitespace-nowrap px-4 py-2 text-gray-400"
							>Volume Down</td
						>
					</tr>

					<tr>
						<td
							class="whitespace-nowrap px-4 py-2 font-medium text-gray-900"
						>
							<kbd>Arrow Right</kbd>
						</td>
						<td class="whitespace-nowrap px-4 py-2 text-gray-400"
							>Next Track</td
						>
					</tr>

					<tr>
						<td
							class="whitespace-nowrap px-4 py-2 font-medium text-gray-900"
						>
							<kbd>Arrow Left</kbd>
						</td>
						<td class="whitespace-nowrap px-4 py-2 text-gray-400"
							>Previous Track</td
						>
					</tr>

					<tr>
						<td
							class="whitespace-nowrap px-4 py-2 font-medium text-gray-900"
						>
							<kbd>S</kbd>
						</td>
						<td class="whitespace-nowrap px-4 py-2 text-gray-400"
							>Shuffle Next</td
						>
					</tr>
					<tr>
						<td
							class="whitespace-nowrap px-4 py-2 font-medium text-gray-900"
						>
							<kbd>L</kbd>
						</td>
						<td class="whitespace-nowrap px-4 py-2 text-gray-400"
							>Loop Mode</td
						>
					</tr>
					<tr>
						<td
							class="whitespace-nowrap px-4 py-2 font-medium text-gray-900"
						>
							<kbd>M</kbd>
						</td>
						<td class="whitespace-nowrap px-4 py-2 text-gray-400"
							>Mute Volume</td
						>
					</tr>

					<tr>
						<td
							class="whitespace-nowrap px-4 py-2 font-medium text-gray-900"
						>
							<kbd>P</kbd>
						</td>
						<td class="whitespace-nowrap px-4 py-2 text-gray-400"
							>Open Playlist</td
						>
					</tr>
					<tr>
						<td
							class="whitespace-nowrap px-4 py-2 font-medium text-gray-900"
						>
							<kbd>C</kbd>
						</td>
						<td class="whitespace-nowrap px-4 py-2 text-gray-400"
							>Clear Playlist</td
						>
					</tr>
					<tr>
						<td
							class="whitespace-nowrap px-4 py-2 font-medium text-gray-900"
						>
							<kbd>T</kbd>
						</td>
						<td class="whitespace-nowrap px-4 py-2 text-gray-400"
							>Tuning</td
						>
					</tr>

					<tr>
						<td
							class="whitespace-nowrap px-4 py-2 font-medium text-gray-900"
						>
							<kbd>H</kbd>
						</td>
						<td class="whitespace-nowrap px-4 py-2 text-gray-400"
							>Macros</td
						>
					</tr>
				</tbody>
			</table>
			<button
				class="bg-black/50 mb-2 ring-1 ring-white/10 ring-inset hover:ring-0 hover:cursor-pointer hover:bg-black/90 border border-white/10 text-xs text-gray-500 font-bold py-1 px-4 rounded-full transition duration-300 ease-in-out shadow-sm shadow-black/50 mx-auto"
				on:click={toggleNotifications}
			>
				{notificationsEnabled
					? "Disable Notifications"
					: "Enable Notifications"}
			</button>
		</div>
	</div>
</div>

<style>
	kbd {
		background-color: #146b0c;
		border-radius: 3px;
		border: 1px solid #5c5a5a;
		box-shadow: 0 1px 1px rgba(0, 0, 0, 0.2),
			0 2px 0 0 rgba(0, 19, 7, 0.7) inset;
		color: #1f1e1e;
		display: inline-block;
		font-size: 1em;
		font-weight: 700;
		line-height: 1;
		padding: 2px 4px;
		white-space: nowrap;
	}
</style>
