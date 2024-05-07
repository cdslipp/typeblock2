<script>
	import Block from './Block.svelte';
	let blocks = $state([{ id: 1, content: '' }]);
	let activeBlockId = $state(null);

	// Listen to global keypresses to navigate blocks or create new ones
	window.addEventListener('keydown', handleKeyDown);

	function handleKeyDown(event) {
		if (event.key === 'Enter') {
			event.preventDefault();
			const newBlockIndex = blocks.findIndex((block) => block.id === activeBlockId) + 1;
			const newBlock = { id: Date.now(), content: '' };
			blocks = [...blocks.slice(0, newBlockIndex), newBlock, ...blocks.slice(newBlockIndex)];
			activeBlockId = newBlock.id;
		}
		// Handle other global shortcuts
	}
</script>

<div>
	{#each blocks as block (block.id)}
		<Block
			key={block.id}
			content={block.content}
			active={block.id === activeBlockId}
			on:activate={() => (activeBlockId = block.id)}
			on:update={(e) => updateBlock(block.id, e.detail.content)}
		/>
	{/each}
</div>

<style>
	/* Your styling */
</style>
