<script>
	import { tweened } from "svelte/motion";
	import { cubicOut } from "svelte/easing";

	export let heightMultiplier;
	export let onPitchToggle = () => {};

	let isExtended = true;

	const tweenedHeight = tweened(1, {
		duration: 800,
		easing: cubicOut,
	});

	// Reactive statement to update the exported value
	$: heightMultiplier = $tweenedHeight;

	// Toggle function
	function toggleHeight() {
		isExtended = !isExtended;
		tweenedHeight.set(isExtended ? 1 : 0);
		onPitchToggle(isExtended);
	}

	// Initialize with extended state
	heightMultiplier = 1;
</script>

<div class="height-toggle">
	<button
		class="toggle-button"
		class:extended={isExtended}
		on:click={toggleHeight}
		aria-pressed={isExtended}
	>
		<span class="button-text">
			{isExtended ? "Collapse" : "Extend"} Height
		</span>
		<span class="button-icon" class:rotated={isExtended}> ↑ </span>
	</button>
</div>
