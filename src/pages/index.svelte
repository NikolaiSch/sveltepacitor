<script lang="ts">
  import { onMount } from "svelte";
  import type { IRootObject } from "./reqTypes";
  import Portal from "svelte-portal/src/Portal.svelte";

  let capitalise = (s) => s[0].toUpperCase() + s.substring(1);

  const POKECOUNT = 300;
  let pokePromise: Promise<IRootObject>;
  let pokeInfo: IRootObject;
  let status = 0;
  let ids = [];

  let getRandPokeId = () => Math.floor(Math.random() * POKECOUNT) + 1;

  let getPokemon = async (id = undefined): Promise<IRootObject> => {
    if (id === undefined) {
      id = getRandPokeId();
      ids.push(id);
    }
    return (await fetch(`https://pokeapi.co/api/v2/pokemon/${id}`)).json();
  };

  onMount(async () => {
    pokePromise = getPokemon();
    pokeInfo = await pokePromise;
    status = 1;
  });
</script>

<div class="flex flex-row justify-evenly mt-1">
  <button
    class="bg-blue-700 text-white px-2 py-1 rounded-sm transition active:scale-125"
    on:click={() => {
      ids.pop();
      pokePromise = getPokemon(ids[ids.length - 1]);
    }}>Previous</button
  >
  <h1 class="my-1">
    PokeName: <span id="pokeName" />
  </h1>
  <button
    on:click={() => {
      pokePromise = getPokemon();
    }}
    class="bg-blue-700 text-white px-2 py-1 rounded-sm transition active:scale-125"
    >Next</button
  >
</div>
{#if status != 0}
  {#await pokePromise then data}
    <Portal target="#pokeName">
      {data.name
        .split("-")
        .map((e) => {
          capitalise(e);
        })
        .join(" ")}
    </Portal>
    <div>
      {#if data.sprites.other["official-artwork"].front_default !== null}
        <img
          src={data.sprites.other["official-artwork"].front_default}
          alt={data.sprites.other["official-artwork"].front_default}
        />
      {/if}
    </div>
  {:catch error}
    <p>{error}</p>
  {/await}
{/if}
