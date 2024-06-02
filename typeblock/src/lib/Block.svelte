<script>
	import { createEventDispatcher } from 'svelte';
	let { value = $bindable(''), active = $bindable(false), block } = $props();
	let editing = $state(false);
	let dispatch = createEventDispatcher();
	let editorElement;

	function onEditorUpdate(event) {
		value = event.target.innerHTML;
		dispatch('valueChange', value);
	}

	function onEditorKeyDown(event) {
		if (event.key === 'Enter') {
			event.preventDefault();
		}
	}

	function onEditorFocus(event) {
		console.log('Focus');
	}

	$effect(() => {
		editing = active;
		if (active) {
			editing = true;
		} else {
			editing = false;
		}

		import('trix')
			.then(() => {
				editorElement = document.querySelector(`trix-editor[data-id="${block.id}"]`);
				if (editorElement) {
					editorElement.addEventListener('trix-change', onEditorUpdate);
					editorElement.addEventListener('keydown', onEditorKeyDown);
					editorElement.addEventListener('focus', onEditorFocus);
				}
			})
			.catch((error) => {
				console.error('Error loading Trix:', error);
			});

		return () => {
			if (editorElement) {
				editorElement.removeEventListener('trix-change', onEditorUpdate);
				editorElement.removeEventListener('keydown', onEditorKeyDown);
				editorElement.removeEventListener('focus', onEditorFocus);
			}
		};
	});

	function handleClick() {
		dispatch('setActiveBlock');
	}
</script>

<div class="block" on:click={handleClick}>
	{#if editing}
		<div class="editorContainer">
			<trix-toolbar id="toolbar"></trix-toolbar>
			<div class="editorWrapper">
				<trix-editor
					toolbar="toolbar"
					input="editor-content"
					class="trix-content editor"
					data-id={block.id}
					autofocus
				/>
				<input id="editor-content" bind:value name="content" type="hidden" />
			</div>
		</div>
	{:else}
		{@html value}
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

	.editorContainer {
		width: 100%;
		height: 100%;
		border: 1px solid #ccc;
	}

	.editorWrapper {
		height: 100%;
	}

	trix-toolbar {
		display: none;
	}

	:global(.trix-content) {
		height: 100%;
		min-height: 100px;
		background: white;
		outline: none;
		line-height: 1.5;
	}

	:global(.trix-content *) {
		box-sizing: border-box;
		margin: 0;
		padding: 0;
	}

	:global(.trix-content h1) {
		font-size: 1.8rem;
		margin: 0;
	}
</style>
