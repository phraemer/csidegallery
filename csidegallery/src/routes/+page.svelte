<script lang="ts">
	import Button, { Label } from '@smui/button';
	import DataTable, { Head, Body, Row, Cell } from '@smui/data-table';
	import Checkbox from '@smui/checkbox';
	import { onMount } from 'svelte';
	import { openDB, initDB, testInsertImages, listImages, type image } from '$lib/SQL.svelte';

	let images: Array<image> = [];

	async function updateImages() {
		try {
			console.log('Listing images');

			images = await listImages();
		} catch (e) {
			console.error('exception selecting data: ', e);
			return;
		}

		console.log('images: ' + JSON.stringify(images));
	}

	onMount(async () => {
		await openDB();
		await initDB();
		await testInsertImages();
	});
</script>

<DataTable style="max-width: 100%;">
	<Head>
		<Row>
			<Cell checkbox>
				<Checkbox />
			</Cell>
			<Cell>Title</Cell>
			<Cell>Description</Cell>
			<Cell>Path</Cell>
		</Row>
	</Head>
	<Body>
		{#each images as image (image.title)}
			<Row>
				<Cell>{image.title}</Cell>
				<Cell>{image.description}</Cell>
				<Cell>{image.path}</Cell>
			</Row>
		{/each}
	</Body>
</DataTable>

<Button on:click={updateImages}>
	<Label>List Images</Label>
</Button>
