<script>
	import { browser } from '$app/environment';
	import Block from './Block.svelte';

	let documentHTML = `<article>
    <section id="introduction">
        <h2>Introduction</h2>
        <p>This software project aims to revolutionize the way we interact with web text editors by introducing a highly structured, block-based approach.</p>
    </section>
    <section id="features">
        <h2>Features</h2>
        <p>The project introduces several key features, including an opinionated text formatting, block management, and real-time collaboration.</p>
    </section>
    <section id="installation">
        <h2>Installation Guide</h2>
        <p>To get started with the project, clone the repository and follow the setup instructions.</p>
    </section>
    <section id="contributing">
        <h2>Contributing</h2>
        <p>Contributions are welcome! Please read the contribution guide to learn how you can contribute to this project.</p>
    </section>
</article>`; // Your HTML document as a string

	let blocks = $state([]);

	function parseHTMLDocument(html) {
		const parser = new DOMParser();
		const doc = parser.parseFromString(html, 'text/html');
		const sections = doc.querySelectorAll('section');
		blocks = Array.from(sections).map((section, index) => {
			return {
				id: index + 1,
				content: section.innerHTML,
				position: index // Add position
			};
		});
	}

	if (browser) {
		parseHTMLDocument(documentHTML);
	}

	let activeBlockId = $state(null);

	function handleKeyDown(event) {
		if (event.key === 'Enter') {
			event.preventDefault();
			const newBlockIndex = blocks.findIndex((block) => block.id === activeBlockId) + 1;
			const newBlock = { id: blocks.length + 1, content: '' };
			blocks = [...blocks.slice(0, newBlockIndex), newBlock, ...blocks.slice(newBlockIndex)];
			activeBlockId = newBlock.id;
		}
	}

	if (browser) {
		window.addEventListener('keydown', handleKeyDown);
	}

	function handleValueChange(id, newValue) {
		const updatedBlocks = blocks.map((block) =>
			block.id === id ? { ...block, content: newValue } : block
		);
		blocks = updatedBlocks;
	}

	function setActiveBlock(id) {
		activeBlockId = id;
	}

	function handleDragStart(event, id) {
		event.dataTransfer.setData('text/plain', id);
		event.dataTransfer.effectAllowed = 'move';
	}

	function handleDragOver(event) {
		event.preventDefault(); // Necessary to allow a drop
		event.dataTransfer.dropEffect = 'move';
	}

	function handleDrop(event, targetId) {
		event.preventDefault();
		const sourceId = +event.dataTransfer.getData('text/plain');
		if (sourceId !== targetId) {
			const sourceIndex = blocks.findIndex((block) => block.id === sourceId);
			const targetIndex = blocks.findIndex((block) => block.id === targetId);
			const [movedBlock] = blocks.splice(sourceIndex, 1);
			blocks.splice(targetIndex, 0, movedBlock);
			blocks = blocks.map((block, index) => ({ ...block, position: index }));
		}
	}
</script>

<div id="typeblock">
	{#if browser}
		{#each blocks as block (block.id)}
			<Block
				{block}
				value={block.content}
				active={block.id === activeBlockId}
				on:valueChange={(e) => handleValueChange(block.id, e.detail)}
				on:setActiveBlock={() => setActiveBlock(block.id)}
			/>
		{/each}
	{/if}
</div>

<style>
	#typeblock {
		display: flex;
		flex-direction: column;
		width: 1000px;
	}
</style>
