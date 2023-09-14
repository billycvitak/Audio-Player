<script>
	import { fly } from 'svelte/transition';
	import { cubicInOut } from 'svelte/easing';
	export let message = '';

	const notificationStyles = `
      position: fixed;
      bottom: 20px;
      padding: 10px 20px;
      background-color: #333;
      color: white;
      border-radius: 5px;
    `;

	let isVisible = false;

	$: {
		isVisible = message !== '';
	}
</script>

{#if isVisible}
	<div
		transition:fly={{ duration: 200, y: 20, easing: cubicInOut }}
		class="shadow-white/30 shadow-md"
		style={notificationStyles}
	>
		{message}
	</div>
{/if}
