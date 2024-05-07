<script>
	import Trix from '$lib/Trix.svelte';

	let text = $state('');
</script>

<h1>Trix Editor Test</h1>
<Trix bind:value={text} />

<div>
	<h2>Output</h2>
	{@html text}

	<h2>Raw HTML</h2>
	{text}
</div>
