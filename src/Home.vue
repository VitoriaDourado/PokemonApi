<template>
  <div class="home">
    <img alt="Poke logo" src="./assets/pokeboll.jpg" width="120" />
    <h1>Escolha um pokemon</h1>
    <input type="text" v-model="searchPoke" placeholder="Nomes...">


    <div class="type-checkboxes">
      <label>
        <input class="type-checkbox" type="checkbox" v-model="searchTypes" value="''" />
        <span></span> Todos
      </label>
      <label v-for="type in pokemonTypes" :key="type.name" :class="[type.name, { 'checked': searchTypes.includes(type.name) }]">
        <input class="type-checkbox" type="checkbox" v-model="searchTypes" :value="type.name" /> 
        <span></span> {{ type.name }}
      </label>
    </div>

    <div class="containerM">
      <div class="card" v-for="(pokemon, i) in filterPoke" :key="i">
        <p>Nomes: {{ pokemon.name }} - ID: {{ getId(pokemon.url) }}</p>
        <p>Tipos: {{ pokemon.types ? pokemon.types.join(', ') : 'Carregando tipos...' }}</p>
        <img class="card img" :src="`${urlBase}${getId(pokemon.url)}.svg`" :alt="pokemon.name" width="100" height="100" />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, computed } from 'vue'
import api from "./services/api";

import cards from "./styles.css"

const pokemons = ref([]);
const searchPoke = ref('');
const searchTypes = ref([]);
const urlBase = ref("https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/dream-world/");

const fetchPokemons = async () => {
  try {
    const totalPokemons = 1307;
    const batchSize = 100;
    let allPokemons = [];

    for (let offset = 0; offset < totalPokemons; offset += batchSize) {
      const response = await api.get(`/pokemon?limit=${batchSize}&offset=${offset}`);
      const results = response.data.results;

      allPokemons = [...allPokemons, ...results];

      await Promise.all(
        results.map(async (pokemon) => {
          const details = await api.get(pokemon.url);
          pokemon.types = details.data.types.map(typeInfo => typeInfo.type.name);
        })
      );
      pokemons.value = [...allPokemons];
    }


  } catch (error) {
    console.error('Erro ao buscar os Pokémons:', error);
  }

};


onMounted(fetchPokemons)


const filterPoke = computed(() => {
  return pokemons.value.filter(pokemon => {
    const pokeName = pokemon.name.toLowerCase().includes(searchPoke.value.toLowerCase());
    const pokeType = searchTypes.value.length === 0 || searchTypes.value.every(type => pokemon.types.includes(type));
    return pokeName && pokeType;
  })
})

const getId = (url: string) => {
  const parteBarra = url.split('/');
  return parteBarra[parteBarra.length - 2];
}

const pokemonTypes = [
  { name: 'normal' },
  { name: 'fire' },
  { name: 'water' },
  { name: 'electric' },
  { name: 'grass' },
  { name: 'ice' },
  { name: 'fighting' },
  { name: 'poison' },
  { name: 'ground' },
  { name: 'flying' },
  { name: 'psychic' },
  { name: 'bug' },
  { name: 'rock' },
  { name: 'ghost' },
  { name: 'dragon' },
  { name: 'dark' },
  { name: 'steel' },
  { name: 'fairy' }
];

</script>


