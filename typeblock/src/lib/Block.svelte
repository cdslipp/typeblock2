<script>
	import Trix from './Trix.svelte';
	import { createEventDispatcher } from 'svelte';

	let { block, content, active = $bindable(false) } = $props();

	let editing = $state(false);

	let dispatch = createEventDispatcher();

	// Emit an activate event when the block becomes active
	$effect(() => {
		editing = active;
		if (active) {
			editing = true;
			// Focus logic here, if necessary
		} else {
			editing = false;
		}
	});

	function exitEditMode() {
		editing = false;
		// Emit update event
	}
	function handleClick() {
		dispatch('setActiveBlock');
	}
</script>

<div class="block" on:click={handleClick}>
	{#if editing}
		<Trix bind:value={block.content} />
	{:else}
		{@html block.content}
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
