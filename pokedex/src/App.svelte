<script>
    import { onMount } from 'svelte';
    let query = '';
    let pokemon = null;
    let error = null;
    let suggestions = [];
    let showSuggestions = false;

    async function fetchPokemon() {
        try {
            const res = await fetch(`https://pokeapi.co/api/v2/pokemon/${query.toLowerCase()}`);
            if (!res.ok) throw new Error('Pokemon not found');
            pokemon = await res.json();
            error = null;
        } catch (err) {
            error = err.message;
            pokemon = null;
        }
    }

    async function fetchSuggestions() {
        try {
            const res = await fetch(`https://pokeapi.co/api/v2/pokemon?limit=1000`);
            const data = await res.json();
            suggestions = data.results.map(p => p.name);
        } catch (err) {           
            console.error('Failed to fetch suggestions', err);
        }
    }

    function handleSearch() {
        if (query) fetchPokemon();
    }

    function handleInput(event) {
        query = event.target.value;
        showSuggestions = query.length > 0;
    }

    function selectSuggestion(suggestion) {
        query = suggestion;
        showSuggestions = false;
        handleSearch();
    }

    onMount(() => {
        fetchSuggestions();
    });
</script>

<style>
    main {
        font-family: 'Press Start 2P', cursive;
        text-align: center;
        padding: 2rem;
        background-color: #ffcc00;
        border: 5px solid #ff0000;
        border-radius: 15px;
        box-shadow: 0 0 20px rgba(0, 0, 0, 0.5);
        max-width: 600px;
        margin: 2rem auto;
    }

    h1 {
        color: #ff0000;
        text-shadow: 2px 2px #000;
    }

    input {
        padding: 0.5rem;
        font-size: 1rem;
        width: 200px;
        border: 2px solid #000;
        border-radius: 5px;
        background-color: #fff;
    }

    button {
        padding: 0.5rem 1rem;
        font-size: 1rem;
        margin-left: 0.5rem;
        cursor: pointer;
        background-color: #ff0000;
        color: white;
        border: 2px solid #000;
        border-radius: 5px;
        text-shadow: 1px 1px #000;
    }

    .suggestions {
        border: 2px solid #000;
        max-height: 150px;
        overflow-y: auto;
        position: absolute;
        background: white;
        width: 200px;
        border-radius: 5px;
        box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
    }

    .suggestions div {
        padding: 0.5rem;
        cursor: pointer;
        border-bottom: 1px solid #ccc;
    }

    .suggestions div:hover {
        background: #ffcc00;
    }

    .pokemon-details {
        display: flex;
        flex-direction: column;
        align-items: center;
        margin-top: 2rem;
        background-color: white;
        padding: 1rem;
        border: 5px solid #ff0000;
        border-radius: 15px;
        box-shadow: 0 0 20px rgba(0, 0, 0, 0.5);
    }

    .pokemon-details img {
        width: 150px;
        height: 150px;
        border: 2px solid #000;
        border-radius: 10px;
    }

    .error {
        color: red;
        margin-top: 1rem;
        font-weight: bold;
    }
</style>

<main>
    <h1>Pokedex</h1>
    <div style="position: relative;">
        <input type="text" bind:value={query} on:input={handleInput} placeholder="Search for a Pokemon..." />
        {#if showSuggestions}
            <div class="suggestions">
                {#each suggestions.filter(s => s.startsWith(query.toLowerCase())).slice(0, 10) as suggestion}
                    <div on:click={() => selectSuggestion(suggestion)} on:keydown={() => {}}>{suggestion}</div>
                {/each}
            </div>
        {/if}
    </div>
    <button on:click={handleSearch}>Search</button>

    {#if error}
        <p class="error">{error}</p>
    {/if}

    {#if pokemon}
        <div class="pokemon-details">
            <h2>{pokemon.name}</h2>
            <img src={pokemon.sprites.front_default} alt={pokemon.name} />
            <p>Height: {pokemon.height}</p>
            <p>Weight: {pokemon.weight}</p>
            <p>Type: {pokemon.types.map(type => type.type.name).join(', ')}</p>
        </div>
    {/if}

    <audio src="/audio/Adding Data to the Pokédex.mp3" autoplay loop></audio>
</main>
