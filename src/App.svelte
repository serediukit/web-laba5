<script>
  import { onMount } from "svelte";
  import http from "./request-helper";
  import OperationDocsStore from "./operationDocsStore";
  import { heroes, token } from "./store";
  import auth from "./auth-service";
  import { isAuthenticated, user } from "./store";
  let auth0Client;
  onMount(async () => {
    auth0Client = await auth.createClient();
    isAuthenticated.set(await auth0Client.isAuthenticated());
    user.set(await auth0Client.getUser());
    if (isAuthenticated) {
      const accessToken = await auth0Client.getIdTokenClaims();
      token.set(accessToken.__raw);
    }
  });

  token.subscribe(async (value) => {
    if (value) {
      const result = await http.startFetchMyQuery(OperationDocsStore.getAll());
      heroes.set(result.heroes);
    }
  });

  function login() {
    auth.loginWithPopup(auth0Client);
  }

  function logout() {
    auth.logout(auth0Client);
  }

  const addHero = async () => {
    const name = prompt("name") || "";
    const desc = prompt("Description") || "";
    const photo = prompt("Photo URL") || "";
    const { insert_heroes_one } = await http.startExecuteMyMutation(
      OperationDocsStore.addOne(name, desc, photo),
    );
    heroes.update((n) => [...n, insert_heroes_one]);
  };

  const deleteHero = async () => {
    const name = prompt("which here to delete?") || "";
    if (name) {
      await http.startExecuteMyMutation(OperationDocsStore.deleteByName(name));
      heroes.update((n) => n.filter((hero) => hero.name !== name));
    }
  };
</script>

<main>
  {#if $isAuthenticated}
    <button on:click={logout}>Log out</button>
    <button on:click={addHero}>Add new hero</button>
    <button on:click={deleteHero}>Delete hero</button>

    {#each $heroes as hero}
      <div>
        <p>Hero name: {hero.name}</p>
        <p>Description: {hero.description}</p>
        <p>Photo:</p>
        {#if hero.img}
          <img src={hero.img} alt={hero.name} width="400" />
        {:else}
          <p>No photo.</p>
        {/if}
        <hr />
      </div>
    {/each}
  {:else}
    <button on:click={login}>Log in</button>
  {/if}
</main>

<style>
  main {
    margin: 0;
  }
</style>
