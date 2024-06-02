<!--TypeBlock.svelte-->
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
	let blocks = $state([{ id: 1, content: 'Test' }]);

	let editorValue = $state('This is the inital value');

	let activeBlockId = $state(null);

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

	// This function is triggered on block click.
	function setActiveBlock(id) {
		activeBlockId = id; // Reactive assignment triggers UI updates.
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
	{#each blocks as block (block.id)}
		<!-- {block.content} -->
		<Block
			bind:value={editorValue}
			key={block.id}
			active={block.id === activeBlockId}
			on:setActiveBlock={() => setActiveBlock(block.id)}
		/>
	{/each}
</div>

<style>
	#typeblock {
		display: flex;
		flex-direction: column;
		width: 1000px;
	}
</style>
