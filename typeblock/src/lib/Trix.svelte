<script>
	let { value = $bindable(), showToolbar = $bindable(true) } = $props();
	let editorElement = null;

	let enterPressCount = $state(0);
	let isFirstClick = $state(true);
	let editingHeading = $state(false);
	let headings = $state();
	let emptyEditor = $state(true);
	let selectionStage = $state(0); // 0: paragraph, 1: section, 2: document

	function stripHtmlAndTrim(htmlContent) {
		return htmlContent.replace(/<[^>]*>/g, '').trim();
	}

	function handleCmdASelection() {
		const [start, end] = editorElement.editor.getSelectedRange();
		console.log(start);
		const htmlContent = editorElement.innerHTML;
		const parser = new DOMParser();
		const doc = parser.parseFromString(htmlContent, 'text/html');
		const allElements = doc.body.childNodes;
		let startElement;
		let cumulativeLength = 0; // Keep track of cumulative text content length

		// Find the start element of the current cursor position
		for (let element of allElements) {
			let elementTextContent = stripHtmlAndTrim(element.innerHTML);
			cumulativeLength += elementTextContent.length;
			console.log('Cumulative length: ', cumulativeLength);
			if (cumulativeLength >= start && !startElement) {
				startElement = element;
				break; // Break once we find the start element
			}
		}

		// Determine the new selection range based on the current stage
		if (selectionStage === 0 && startElement && !editingHeading) {
			console.log(startElement);
			// Select the current paragraph
			let paragraphStart = cumulativeLength - stripHtmlAndTrim(startElement.innerHTML).length;
			editorElement.editor.setSelectedRange([
				paragraphStart,
				paragraphStart + stripHtmlAndTrim(startElement.innerHTML).length
			]);
			selectionStage = 1;
		} else if (selectionStage === 1) {
			// Select the current section (up to the closest prior heading)
			let sectionStart = 0;
			for (let element of allElements) {
				if (element === startElement) break;
				if (element.tagName === 'H1') sectionStart = cumulativeLength;
				else cumulativeLength += stripHtmlAndTrim(element.innerHTML).length;
			}
			editorElement.editor.setSelectedRange([sectionStart, cumulativeLength]);
			selectionStage = 2;
		} else {
			// Select the whole document
			editorElement.editor.setSelectedRange([0, cumulativeLength]);
			selectionStage = 0;
		}
	}

	function extractHeadings(htmlContent) {
		const parser = new DOMParser();
		const doc = parser.parseFromString(htmlContent, 'text/html');
		const h1Elements = doc.querySelectorAll('h1');
		const extractedHeadings = Array.from(h1Elements).map((h1, index) => ({
			id: `heading-${index}`,
			text: h1.textContent
		}));
		headings = extractedHeadings;
	}

	function startFirstHeading() {
		if (!editingHeading) {
			editingHeading = true;
			editorElement.editor.setSelectedRange([0, 0]);
			editorElement.editor.activateAttribute('heading1');
			editorElement.editor.setSelectedRange([0, 0]);
		}
	}

	function onEditorUpdate() {
		if (editorElement) {
			const htmlContent = editorElement.innerHTML;
			value = htmlContent;

			// Remove all HTML tags and check if any text remains
			const textContent = stripHtmlAndTrim(htmlContent);

			if (!textContent && !emptyEditor) {
				console.log('Empty');
				editorElement.editor.loadHTML('');
				startFirstHeading();
				emptyEditor = true; // Set the editor as empty
			} else if (textContent) {
				emptyEditor = false; // Set the editor as not empty
			}
		}
	}

	function onEditorKeyDown(event) {
		if (event.key === 'Enter') {
			event.preventDefault();

			if (event.shiftKey) {
				// Shift + Enter: Single line break
				editorElement.editor.insertLineBreak();
			} else if (event.metaKey || event.ctrlKey) {
				// Cmd + Enter or Ctrl + Enter: Two line breaks and heading mode
				editorElement.editor.insertLineBreak();
				editorElement.editor.insertLineBreak();
				editorElement.editor.activateAttribute('heading1');
				editingHeading = true;
			} else if (editingHeading) {
				// Enter while editing a heading: Single line break
				editorElement.editor.insertLineBreak();
				editingHeading = false;
			} else {
				// Regular Enter key behavior
				enterPressCount++;

				switch (enterPressCount) {
					case 1:
						if (!editingHeading) {
							editorElement.editor.insertLineBreak();
							editorElement.editor.insertLineBreak();
						}
						break;
					case 2:
						editorElement.editor.activateAttribute('heading1');
						editingHeading = true;
						break;
					case 3:
						editorElement.editor.setSelectedRange([0, 9]);
						enterPressCount = 0; // Reset the count
						break;
				}
			}
		} else if (event.metaKey && event.key === 'a') {
			event.preventDefault();
			handleCmdASelection();
		} else {
			enterPressCount = 0; // Reset the count if any other key is pressed
			selectionStage = 0;
		}
	}

	function onEditorFocus() {
		if (isFirstClick) {
			editorElement.editor.activateAttribute('heading1');
			editingHeading = true;
			isFirstClick = false;
		}
	}

	$effect(() => {
		import('trix')
			.then(() => {
				editorElement = document.querySelector('trix-editor');
				if (editorElement) {
					editorElement.addEventListener('trix-change', onEditorUpdate);
					editorElement.addEventListener('keydown', onEditorKeyDown);
					editorElement.addEventListener('focus', onEditorFocus);
				}
			})
			.catch((error) => {
				console.error('Error loading Trix:', error);
			});
		extractHeadings(value);

		return () => {
			if (editorElement) {
				editorElement.removeEventListener('trix-change', onEditorUpdate);
				editorElement.removeEventListener('keydown', onEditorKeyDown);
				editorElement.removeEventListener('focus', onEditorFocus);
			}
		};
	});
</script>

<div class="container">
	<nav class="toc">
		<h1>Document Outline</h1>
		{#if headings}
			{#each headings as heading}
				<h3>{heading.text}</h3>
			{/each}
		{/if}
	</nav>
	<div class="editorContainer">
		<trix-toolbar id="toolbar"></trix-toolbar>
		<div class="editorWrapper">
			<trix-editor toolbar="toolbar" input="editor-content" class="trix-content editor" />
		</div>
	</div>
</div>

<style>
	.container {
		display: grid;
		grid-template-columns: 1fr 3fr;
		gap: 1rem;
	}

	.editorContainer {
		display: flex;
		flex-direction: column;
	}

	.editorWrapper {
		border: 1px solid black;
		overflow: scroll;
		border-radius: 4px;
		box-shadow: 0 1px 6px grey;
		padding: 12px;
		margin-top: 1rem;
		margin-bottom: 1rem;
		height: 50dvh;
	}

	:global(.trix-content) {
		height: 100%;
		outline: none; /* Removes the blue outline */
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

	.toc {
		/* Styles for the Table of Contents */
		border-right: 1px solid var(--secondary-color);
		padding: 1rem;
	}
</style>
