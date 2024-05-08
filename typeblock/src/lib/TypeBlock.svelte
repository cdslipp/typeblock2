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
</article>`; // Your HTML document as string
	let blocks = $state([]);

	let activeBlockId = blocks[0]?.id;

	$effect(() => {
		parseHTMLDocument(documentHTML);
	});

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
		// Existing window event listener logic here...
	}
	if (browser) {
		// Listen to global keypresses to navigate blocks or create new ones
		window.addEventListener('keydown', handleKeyDown);
	}

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
