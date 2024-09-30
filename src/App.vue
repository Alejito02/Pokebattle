<template>
  <div class="app-container">
    <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/9/98/International_Pok%C3%A9mon_logo.svg/640px-International_Pok%C3%A9mon_logo.svg.png" alt="Pokémon Logo" class="pokemon-logo" />

    <div class="card-container">
      <div class="card" v-for="(pokemon, index) in pokemons" :key="index" @click="selectPokemon(pokemon)">
        <div class="pokemon-image-container" :style="{ backgroundColor: typeColors[pokemon.types[0].type.name] }">
          <img :src="pokemon.sprites?.front_default" alt="Pokemon Image" class="pokemon-image" />
        </div>
        
        <h4>{{ pokemon.name ? pokemon.name || capitalize : 'Nombre no disponible' }}</h4>
        
        <div class="types">
          <button 
            v-for="type in pokemon.types" 
            :key="type.slot" 
            :style="{ backgroundColor: typeColors[type.type.name] }"
            class="type-button"
          >
            {{ type.type.name }}
          </button>
        </div>

        <div class="stats">
          <h6>Estadísticas</h6>
          <ul>
            <li v-for="stat in pokemon.stats.filter(stat => ['hp', 'attack', 'defense'].includes(stat.stat.name))" :key="stat.stat.name">
              <div class="stat-name">{{ stat.stat.name || capitalize }}: {{ stat.base_stat }}</div>
              <div class="stat-bar">
                <div 
                  class="stat-bar-fill" 
                  :style="{ width: (stat.base_stat / 225 * 100) + '%' }"
                ></div>
              </div>
            </li>
          </ul>
        </div>
      </div>
      <div class="battle-button-container">
        <button @click="battle">Batalla!</button>
      </div>
      <div v-if="winner" class="winner-card">
        <h4>El ganador es {{ winner.name }}!</h4>
      </div>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue';

export default {
  setup() {
    const pokemons = ref([]);
    const winner = ref(null);
    const selectedPokemon = ref(null);

    const typeColors = ref({
      normal: '#A8A878',
      fire: '#F08030',
      water: '#6890F0',
      electric: '#F8D030',
      grass: '#78C850',
      ice: '#98D8D8',
      fighting: '#C03028',
      poison: '#A040A0',
      ground: '#E0C068',
      flying: '#A890F0',
      psychic: '#F85888',
      bug: '#A8B820',
      rock: '#B8A038',
      ghost: '#705898',
      dragon: '#7038F8',
      dark: '#705848',
      steel: '#B8B8D0',
      fairy: '#EE99AC',
    });

    const capitalize = (word) => {
      return word.charAt(0).toUpperCase() + word.slice(1);
    };

    const getRandomPokemonId = () => {
      return Math.floor(Math.random() * 1010) + 1;
    };

    const fetchPokemonData = async (id) => {
      try {
        const response = await fetch(`https://pokeapi.co/api/v2/pokemon/${id}`);
        const data = await response.json();
        pokemons.value.push(data);
      } catch (error) {
        console.error(error);
      }
    };

    const battle = async () => {
      const pokemon1 = pokemons.value[0];
      const pokemon2 = pokemons.value[1];

      const pokemon1Stats = pokemon1.stats.reduce((acc, stat) => acc + stat.base_stat, 0);
      const pokemon2Stats = pokemon2.stats.reduce((acc, stat) => acc + stat.base_stat, 0);

      if (pokemon1Stats > pokemon2Stats) {
        winner.value = pokemon1;
      } else if (pokemon2Stats > pokemon1Stats) {
        winner.value = pokemon2;
      } else {
        winner.value = null;
      }
    };

    const selectPokemon = (pokemon) => {
      selectedPokemon.value = pokemon;
    };

    fetchPokemonData(getRandomPokemonId());
    fetchPokemonData(getRandomPokemonId());

    return {
      pokemons,
      winner,
      typeColors,
      capitalize,
      battle,
      selectPokemon,
    };
  },
};
</script>

<style>
.app-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: 10px;
  font-family: 'Arial', sans-serif;
  background-image: url("https://w.wallhaven.cc/full/4l/wallhaven-4l6o.jpg");
  background-size: cover;
  background-position: center;
  background-attachment: fixed;
  height: 100vh;
  width: 100vw;
  overflow: hidden;
  text-align: center;
}

.card-container {
  display: flex;
  flex-direction: row;
  justify-content: center;
  gap: 20px;
  position: relative;
}

.card {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  background-color: #fff;
  width: 300px;
  cursor: pointer;
}

.pokemon-logo {
  width: 200px;
  margin: 20px;
}

.pokemon-image-container {
  width: 100px;
  height: 100px;
  border-radius: 50%;
  margin: 20px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.pokemon-image {
  width: 80px;
  height: 80px;
}

.types {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin: 20px;
}

.type-button {
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 16px;
}

.stats {
  margin: 20px;
}

.stat-name {
  font-size: 16px;
  font-weight: bold;
}

.stat-bar {
  width: 100%;
  height: 10px;
  background-color: #ccc;
  border-radius: 5px;
  margin: 10px 0;
}

.stat-bar-fill {
  height: 100%;
  background-color: #4CAF50;
  border-radius: 5px;
}

.battle-button-container {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 1;
}

.battle-button-container button {
  padding: 10px 20px;
  background-color: #ffcc00;
  border: none;
  cursor: pointer;
  border-radius: 5px;
  font-size: 16px;
}

.winner-card {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 1;
  background-color: #fff;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.stats,h6{
  color: black;
}
.card,h4{
  color: black;
}
</style>