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
				content: section.innerHTML
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

	$effect(() => {
		if (activeBlockId !== null) {
			const activeBlockIndex = blocks.findIndex((block) => block.id === activeBlockId);
			if (activeBlockIndex !== -1) {
				blocks[activeBlockIndex].content = editorValue;
			}
		}
	});
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
