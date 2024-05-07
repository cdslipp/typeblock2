<script>
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
	});

	function exitEditMode() {
		editing = false;
		// Emit update event
	}

	$effect(() => {
		if (!active && editing) {
			exitEditMode();
		}
	});
</script>

{#if editing}
	<!-- Active Trix Editor instance here -->
	<trix-editor ... />
{:else}
	<!-- Display static text content -->
	<div on:click={() => (active = true)}>{content}</div>
{/if}

<style>
	/* Your styling */
</style>
