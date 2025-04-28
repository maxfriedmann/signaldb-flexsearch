<script lang="ts">
  import { Collection } from '@signaldb/core';
  import svelteReactivityAdapter from "@signaldb/svelte";
  import { Index, Document, Encoder, Charset, Resolver, Worker } from "flexsearch";
  import { faker } from '@faker-js/faker';

  const logs = $state([]);

  // create posts collection
  const posts = new Collection({ name: 'posts', reactivity: svelteReactivityAdapter });

  // create index
  const index = new Document({
    document: {
      id: "id",
      store: true,
      index: ["title", "text"]
    },
    tokenize: "full",
    encoder: Charset.LatinBalance
  });

  // add collection events
  posts.on("added", (e) => {
    index.add(e.id, e)
    logs.push("Added to search index: " + e.id);
  });

  // fill collection
  for (let i = 0; i < 5; i++){
    posts.insert({ title: faker.company.buzzNoun(), text: faker.company.buzzPhrase(), likes: faker.number.int(100) }); 
  }

  // methods
  let searchText = $state("");
  let searchResults = $state(undefined);
  function search() {
    searchResults = index.search({query: searchText, suggest: true, enrich: true, highlight: "<span class='text-yellow-200'>$1</span>" });
  }
 
</script>
 
<div class="flex flex-row gap-1">
  <input type="text" class="input input-border" bind:value={searchText}>
  <button class="btn btn-primary" onclick={search}>Search</button>
</div>
{#if searchResults}
  <div class="flex flex-col">
    {#each searchResults as searchResult }
      <pre>{JSON.stringify(searchResult)}</pre>
    {/each}
  </div>
{/if}

<div class="text-lg">Data</div>
<table class="table table-zebra">
<thead>
  <tr>
    <th>ID</th>
    <th>Title</th>
    <th>Text</th>
    <th>Likes</th>
  <tr>
</thead>
<tbody>
{#each posts.find().fetch() as post}
  <tr>
    <td>{post.id}</td>
    <td>{post.title}</td>
    <td>{post.text}</td>
    <td>{post.likes}</td>
    </tr>
    {/each}
  </tbody>
</table>

<div class="text-lg">Logs</div>
  {#each logs as log}
    <pre>{log}</pre>
  {/each}