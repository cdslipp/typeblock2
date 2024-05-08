<script>
	import Trix from './Trix.svelte';

	let { content, active = false } = $props();

	let editing = $state(false);

	// Emit an activate event when the block becomes active
	$effect(() => {
		if (active && !editing) {
			editing = true;
			setTimeout(() => {
				// Focus logic here
			});
		}
		if (!active && editing) {
			exitEditMode();
		}
	});

	function exitEditMode() {
		editing = false;
		// Emit update event
	}

	function handleBlockClick() {
		console.log('Block was clicked');
		active = true;
	}
</script>

<div class="block" on:click={handleBlockClick}>
	{#if editing}
		<!-- Active Trix Editor instance here -->
		<Trix bind:value={content} />
	{:else}
		<!-- Display static text content -->
		<div>{@html content}</div>
	{/if}
</div>

<style>
	.block {
		background: grey;
		height: 100%;
		min-height: 100px;
		width: 100%;
		display: flex;
		flex-direction: column;
		margin: 1rem;
	}
</style>
