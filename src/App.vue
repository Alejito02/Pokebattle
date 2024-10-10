<template>
  <div class="app-container">
    <div v-if="!gameStarted" class="welcome-screen">
      <img src="https://images-wixmp-ed30a86b8c4ca887773594c2.wixmp.com/f/e8ddc4da-23dd-4502-b65b-378c9cfe5efa/dfgdnvd-2ca9ea08-d95a-4a0e-bddf-d956e091a924.png/v1/fill/w_1280,h_1280/pokemon_battle_revolution_logo_by_jormxdos_dfgdnvd-fullview.png?token=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJ1cm46YXBwOjdlMGQxODg5ODIyNjQzNzNhNWYwZDQxNWVhMGQyNmUwIiwiaXNzIjoidXJuOmFwcDo3ZTBkMTg4OTgyMjY0MzczYTVmMGQ0MTVlYTBkMjZlMCIsIm9iaiI6W1t7ImhlaWdodCI6Ijw9MTI4MCIsInBhdGgiOiJcL2ZcL2U4ZGRjNGRhLTIzZGQtNDUwMi1iNjViLTM3OGM5Y2ZlNWVmYVwvZGZnZG52ZC0yY2E5ZWEwOC1kOTVhLTRhMGUtYmRkZi1kOTU2ZTA5MWE5MjQucG5nIiwid2lkdGgiOiI8PTEyODAifV1dLCJhdWQiOlsidXJuOnNlcnZpY2U6aW1hZ2Uub3BlcmF0aW9ucyJdfQ.oLSOZ2nlDXjLOfuFiEtT8cSYr3k7WrWfhAL_SYgWrYY" class="logo-pokemon">
      <label for="rounds" class="round-txt">¿Cuántas rondas quieres jugar?</label>
      <input v-model="rounds" type="number" min="1" id="rounds" />
      <button @click="startGame">Comenzar Juego</button>
    </div>

    <div v-if="gameStarted" class="game-container">
      <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/9/98/International_Pok%C3%A9mon_logo.svg/640px-International_Pok%C3%A9mon_logo.svg.png" alt="Pokémon Logo" class="pokemon-logo" />
      
      <div v-if="!currentStat && currentRound < rounds" class="stat-selection">
        <h3>Selecciona una estadística para combatir:</h3>
        <button v-for="stat in ['hp', 'attack', 'defense']" :key="stat" @click="selectStat(stat)">
          {{ capitalize(stat) }}
        </button>
      </div>

      <div v-else>
        <div class="card-row">
          <div class="card" v-for="(pokemon, index) in currentPokemons" :key="index">
            <div class="pokemon-image-container" :style="{ backgroundColor: typeColors[pokemon.types[0].type.name] }">
              <img :src="pokemon.sprites?.front_default" alt="Pokemon Image" class="pokemon-image" />
            </div>
            <h3>{{ pokemon.name ? pokemon.name : 'Nombre no disponible' }}</h3>
            <div class="stats">
              <h5>Estadísticas</h5>
              <ul>
                <li v-for="stat in pokemon.stats.filter(stat => ['hp', 'attack', 'defense'].includes(stat.stat.name))" :key="stat.stat.name">
                  <div class="stat-name">{{ capitalize(stat.stat.name) }}: {{ stat.base_stat }}</div>
                </li>
              </ul>
            </div>
          </div>
        </div>
        
        <div v-if="currentRound < rounds" class="battle-button-container">
          <button @click="battle">Batalla!</button>
        </div>

        <div v-if="winner" class="winner-card">
          <h3>Ganador de la ronda: {{ winner.name }}</h3>
        </div>
      </div>

      <div class="scoreboard">
        <h4>Rondas Jugadas: {{ currentRound }}/{{ rounds }}</h4>
        <h4>Puntuaciones:</h4>
        <p>Jugador 1: {{ player1Score }}</p>
        <p>Jugador 2: {{ player2Score }}</p>
      </div>

      <div v-if="currentRound >= rounds" class="final-winner">
        <h2>El ganador final es: {{ finalWinner }}</h2>
        <button @click="resetGame">Volver a Jugar</button>
      </div>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue';

export default {
  setup() {
    const gameStarted = ref(false);
    const rounds = ref(1);
    const currentRound = ref(0);
    const player1Score = ref(0);
    const player2Score = ref(0);
    const currentPokemons = ref([]);
    const winner = ref(null);
    const finalWinner = ref(null);
    const currentStat = ref(null);

    const typeColors = ref({
      normal: '#A8A878',
      fire: '#F08030',
      water: '#6890F0',
      electric: '#F8D030',
      grass: '#78C850',
      // ... (otros colores)
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
        return data;
      } catch (error) {
        console.error(error);
      }
    };

    const startGame = () => {
      gameStarted.value = true;
      currentRound.value = 0;
      player1Score.value = 0;
      player2Score.value = 0;
      finalWinner.value = null;
      nextRound();
    };

    const nextRound = async () => {
      if (currentRound.value >= rounds.value) return;
      currentRound.value += 1;
      currentStat.value = null;
      const pokemon1 = await fetchPokemonData(getRandomPokemonId());
      const pokemon2 = await fetchPokemonData(getRandomPokemonId());
      currentPokemons.value = [pokemon1, pokemon2];
    };

    const selectStat = (stat) => {
      currentStat.value = stat;
    };

    const battle = () => {
      const [pokemon1, pokemon2] = currentPokemons.value;
      const stat1 = pokemon1.stats.find(stat => stat.stat.name === currentStat.value).base_stat;
      const stat2 = pokemon2.stats.find(stat => stat.stat.name === currentStat.value).base_stat;

      if (stat1 > stat2) {
        winner.value = pokemon1;
        player1Score.value += 1;
      } else if (stat2 > stat1) {
        winner.value = pokemon2;
        player2Score.value += 1;
      } else {
        player1Score.value += 1;
        player2Score.value += 1;
      }

      if (currentRound.value < rounds.value) {
        setTimeout(() => {
          nextRound();
        }, 2000);
      } else {
        determineFinalWinner();
      }
    };

    const determineFinalWinner = () => {
      if (player1Score.value > player2Score.value) {
        finalWinner.value = "Jugador 1";
      } else if (player2Score.value > player1Score.value) {
        finalWinner.value = "Jugador 2";
      } else {
        finalWinner.value = "Empate";
      }
    };

    const resetGame = () => {
      gameStarted.value = false;
      rounds.value = 1;
      currentRound.value = 0;
      player1Score.value = 0;
      player2Score.value = 0;
      winner.value = null;
      finalWinner.value = null;
    };

    return {
      gameStarted,
      rounds,
      currentRound,
      player1Score,
      player2Score,
      currentPokemons,
      winner,
      finalWinner,
      currentStat,
      typeColors,
      capitalize,
      startGame,
      selectStat,
      battle,
      resetGame,
    };
  },
};
</script>

<style>
body {
  margin: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  background-image: linear-gradient(rgba(231, 225, 225, 0.301), rgba(5, 7, 12, 0.226)), url(https://images.squarespace-cdn.com/content/v1/55ef0e29e4b099e22cdc9eea/1456900755158-471EAAKS7VVIHZZPHU9Y/image-asset.jpeg?format=1500w);
  background-color: transparent;
  background-repeat: no-repeat;
  align-content: center;
}

.logo-pokemon{
  max-width: 400px;
}


.rounds{
  size: 10%;
}

.round-txt{
 size: 5%;
}
.app-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
  color: black;
  background-image: url("");
  background-size: 100%;
  background-repeat: no-repeat;
}

#app{
  padding: 0%;
}

.game-container{
background: rgb(255,7,7);
background: linear-gradient(13deg, rgba(255,7,7,0.8491771708683473) 0%, rgba(0,5,255,0.7931547619047619) 100%);
}

.pokemon-logo {
  width: 200px;
  margin: 20px;
}

.card-row {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 20px;
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

.battle-button-container {
  margin-top: 20px;
}

.winner-card {
  margin-top: 20px;
}

.scoreboard, .final-winner {
  margin-top: 20px;
}

.stat-selection button {
  padding: 10px 20px;
  margin: 10px;
  background-color: #ffcc00;
  border: none;
  cursor: pointer;
  border-radius: 5px;
  font-size: 16px;
}

.final-winner button {
  margin-top: 20px;
  padding: 10px 20px;
  background-color: #ff5733;
  border: none;
  cursor: pointer;
  border-radius: 5px;
  font-size: 16px;
  color: white;
}

.welcome-screen {
  display: flex;
  flex-direction: column;
  align-items: center;
  height: 100vh; 
  width: 100%;
}

@media screen and (max-width:600px){

.logo-pokemon{
  max-width: 200px;
}
.card-row{
  display: flex;
  flex-direction: row;
}
.card {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 10px; 
  border: 1px solid #ccc;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  background-color: #fff;
  width: 200px; 
}

.pokemon-image-container {
  width: 80px; 
  height: 80px;
  border-radius: 50%;
  margin: 10px; 
  display: flex;
  justify-content: center;
  align-items: center;
}

.pokemon-image {
  width: 60px; 
  height: 60px;
}

.card h3 {
  font-size: 16px; 
  margin: 10px 0;
}

.stats h5 {
  font-size: 14px; 
}

.stats ul li .stat-name {
  font-size: 12px; 
}

.battle-button-container button {
  padding: 8px 16px; 
  font-size: 14px;   
}

}
</style>
