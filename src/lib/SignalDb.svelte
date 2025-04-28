<script lang="ts">
  import { faker } from "@faker-js/faker";
  import { Collection } from "@signaldb/core";
  import svelteReactivityAdapter from "@signaldb/svelte";
  import { Charset, Document } from "flexsearch";

  const logs: string[] = $state([]);
  const logsReversed = $derived([...logs].reverse());

  // create posts collection
  const posts = new Collection({
    name: "posts",
    reactivity: svelteReactivityAdapter,
  });

  // create index
  const index = new Document({
    document: {
      id: "id",
      store: true,
      index: ["title", "text"],
    },
    tokenize: "full",
    encoder: Charset.LatinBalance,
  });

  // add collection events
  posts.on("added", (e) => {
    index.add(e.id, e);
    logs.push("Added to search index: " + e.id);
  });
  posts.on("changed", (e) => {
    index.update(e.id, e);
    logs.push("Updated search index: " + e.id);
  });

  // fill collection
  for (let i = 0; i < 500; i++) {
    posts.insert({
      title: faker.company.buzzNoun(),
      text: faker.company.buzzPhrase(),
      likes: faker.number.int(100),
    });
  }

  // randomize
  function randomize() {
    posts.find().forEach((post) => {
      posts.updateOne(
        { id: post.id },
        {
          $set: {
            title: faker.company.buzzNoun(),
            text: faker.company.buzzPhrase(),
            likes: faker.number.int(100),
          },
        },
      );
    });
  }

  // search
  let searchText = $state("");
  let searchResults: any[] = $state([]);
  function search(text: string) {
    searchResults = (
      index.search({
        query: text,
        suggest: true,
        enrich: true,
        highlight: "<span class='text-yellow-200'>$1</span>",
      }) as any[]
    ).reduce((p, c) => {
      return [...p, ...c.result];
    }, []);
  }
  $effect(() => {
    search(searchText);
  });
</script>

<div class="text-3xl py-4">Search</div>
<div class="flex flex-row gap-1">
  <input type="text" class="input input-border" bind:value={searchText} />
</div>
{#if searchResults}
  <div class="flex flex-col">
    {#each searchResults as searchResult}
      <div class="flex flex-row gap-1">
        ID: {searchResult.id} - Found: {@html searchResult.highlight}
      </div>
    {/each}
  </div>
{/if}

<div class="text-3xl py-4">Data</div>
<div class="italic">Showing max. 10 of {posts.find().count()}</div>
<table class="table table-zebra">
  <thead>
    <tr>
      <th>ID</th>
      <th>Title</th>
      <th>Text</th>
      <th>Likes</th>
    </tr><tr> </tr></thead
  >
  <tbody>
    {#each posts.find({}, { limit: 10 }).fetch() as post}
      <tr>
        <td>{post.id}</td>
        <td>{post.title}</td>
        <td>{post.text}</td>
        <td>{post.likes}</td>
      </tr>
    {/each}
  </tbody>
</table>
<button class="btn btn-primary" onclick={randomize}>Randomize Values</button>

<div class="text-3xl py-4">Logs</div>
{#each logsReversed as log}
  <pre>{log}</pre>
{/each}
