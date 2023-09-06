<script lang="ts">
  import Button, { Label } from '@smui/button';
  import DataTable, { Head, Body, Row, Cell } from '@smui/data-table';
  import Checkbox from '@smui/checkbox';
  import { onMount } from 'svelte';
  import { openDB, initDB, insertImages, listImages, type image } from '$lib/SQL.svelte';

  let images: Array<image> = [];

  async function updateImages() {
    try {
      console.log('Listing images');

      images = await listImages(50);
    } catch (e) {
      console.error('exception selecting data: ', e);
      return;
    }

    console.log('images: ' + JSON.stringify(images));
  }

  async function openImageList() {
    const fileSelector = document.createElement('input');
    fileSelector.setAttribute('type', 'file');
    fileSelector.setAttribute('multiple', 'multiple');
    fileSelector.setAttribute('accept', 'text/*');
    fileSelector.addEventListener('change', async (event) => {
      if (event.target === null) {
        console.error('event.target is null');
        return;
      }

      // Ignore type error on event.target.files
      // @ts-ignore
      console.log(JSON.stringify(event.target.files));

      // Read each file into a string
      // @ts-ignore
      for (const file of event.target.files) {
        const reader = new FileReader();
        reader.onload = async (event) => {
          if (event.target === null) {
            console.error('event.target is null');
            return;
          }

          // Ignore type error on event.target.result
          // @ts-ignore
          const fileContents: string = event.target.result;
          console.log('fileContents: ' + fileContents);

          // Split the file contents by line into an array of strings
          const lines: Array<string> = fileContents.split('\n');
          // Iterate over the lines
          for (const line of lines) {
            // Here is an example line
            // ./2014/01/11875-Esquimau-Children-in-Esquimau-Village-on-the-Pike-Worlds-Fair-St.-Louis-Mo-800x404.jpg
            // Extract the file name from the path
            let fileName: string = line.substring(line.lastIndexOf('/') + 1);
            console.log('fileName: ' + fileName);

            // Remove the file extension and convert dashes to spaces
            let title: string = fileName.substring(0, fileName.lastIndexOf('.')).replace(/-/g, ' ');

            // If the title starts with a number, remove it, adding it to tag if it is 4 digits long
            let tag: string = '';
            if (title.match(/^\d{4}/)) {
              tag = title.substring(0, 4);
              title = title.substring(5);
            }

            // If the title ends with a number followed by an "x" followed by another number, remove it
            if (title.match(/\d+x\d+$/)) {
              title = title.substring(0, title.lastIndexOf(' '));
            }

            console.log('title: ' + title);

            // Insert the image into the database
            await insertImages([{ path: line, title: title, description: '' }]);
          }
        };
        reader.readAsText(file);
      }
    });
    fileSelector.click();
  }

  onMount(async () => {
    await openDB();
    await initDB();
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
    {#each images as image}
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

<!-- A button that opens a file selection dialog -->
<Button on:click={openImageList}>
  <Label>Upload Images</Label>
</Button>
