<template>
  <div class="app">
    <div v-if="!isBattleStarted" class="home-page">
      <img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTxSBKFSUdL3VxbQXdSltIaE13yS4LT-df1Cw&s" class="Imagetittle">
      <h3>Ingresa tu nombre y elige el número de rondas para empezar la batalla</h3>
      <div class="input-container">
        <input
          v-model="playerName"
          placeholder="Ingresa tu nombre"
          @keyup.enter="startBattle"
          class="styled-input"
        />
        <select v-model="totalRounds" class="styled-select">
          <option disabled value="">Selecciona el número de rondas</option>
          <option v-for="round in [3, 5, 7]" :key="round" :value="round">
            {{ round }}
          </option>
        </select>
        <button @click="startBattle" class="styled-button">Iniciar Batalla</button>
      </div>
    </div>

    <div v-else class="battle-page">
      <h2>{{ playerName }} vs Bot</h2>
      <p>Ronda actual: {{ currentRound }} / {{ totalRounds }}</p>
      <div class="battle-field">
        <div class="player-side">
          <h3>{{ playerName }}</h3>
          <div v-if="playerTeam.length" class="pokemon-battle">
            <img :src="playerTeam[0].backImage" :alt="playerTeam[0].name" class="pokemon-back" />
          </div>
        </div>
        <div class="bot-side">
          <h3>Bot</h3>
          <div v-if="botTeam.length" class="pokemon-battle">
            <img :src="botTeam[0].image" :alt="botTeam[0].name" class="pokemon-front" />
          </div>
        </div>
      </div>
      <div class="input-container">
        <select v-model="selectedStat" class="styled-select">
          <option disabled value="">Selecciona una característica</option>
          <option value="hp">HP</option>
          <option value="attack">Ataque</option>
          <option value="defense">Defensa</option>
        </select>
        <button @click="playRound" class="styled-button">Jugar Ronda</button>
      </div>
      <div v-if="roundResult">
        <p>{{ roundResult }}</p>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'App',
  data() {
    return {
      playerName: '',
      isBattleStarted: false,
      totalRounds: '',
      currentRound: 1,
      playerTeam: [],
      botTeam: [],
      selectedStat: '',
      roundResult: '',
      playerWins: 0,
      botWins: 0,
    };
  },
  methods: {
    startBattle() {
      if (this.playerName.trim() && this.totalRounds) {
        this.isBattleStarted = true;
        this.fetchPokemon();
      }
    },
    async fetchPokemon() {
      try {
        const pokemonIds = Array.from({ length: 6 }, () =>
          Math.floor(Math.random() * 898) + 1
        );
        const promises = pokemonIds.map((id) =>
          axios.get(`https://pokeapi.co/api/v2/pokemon/${id}`)
        );
        const responses = await Promise.all(promises);
        const pokemons = responses.map((response) => ({
          name: response.data.name,
          image: response.data.sprites.front_default,
          backImage: response.data.sprites.back_default,
          stats: {
            hp: response.data.stats.find((s) => s.stat.name === 'hp').base_stat,
            attack: response.data.stats.find((s) => s.stat.name === 'attack').base_stat,
            defense: response.data.stats.find((s) => s.stat.name === 'defense').base_stat,
          },
        }));
        this.playerTeam = pokemons.slice(0, 3);
        this.botTeam = pokemons.slice(3, 6);
      } catch (error) {
        console.error("Error al obtener los Pokémon:", error);
      }
    },
  },
};
</script>

<style scoped>
.app {
  width: 100%;
  text-align: center;
  background-image: url("https://www.pngkit.com/png/full/7-70236_pokemon-battle-background-png-pokemon-battle-background-hd.png");
  background-size: cover;
  height: 100vh;
  font-family: 'Arial', sans-serif;
  color: white;
  display: flex;
  justify-content: center;
  align-items: center;
}

.battle-field {
  display: flex;
  justify-content: space-between;
  width: 100%;
  max-width: 600px;
  position: relative;
  margin: auto;
}

.player-side, .bot-side {
  width: 50%;
  text-align: center;
}

.pokemon-battle img {
  width: 120px;
  height: 120px;
}

.pokemon-back {
  transform: scaleX(-1);
}

.input-container {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.styled-input, .styled-select, .styled-button {
  padding: 12px;
  margin: 8px;
  border-radius: 12px;
  border: 2px solid white;
  font-size: 18px;
  background-color: rgba(255, 255, 255, 0.2);
  color: white;
  text-shadow: 1px 1px black;
  width: 300px;
}

.styled-input {
  text-align: center;
}

.styled-select {
  background-color: #ffcc00;
  color: black;
}

.styled-button {
  background-color: #ff5733;
  color: white;
  cursor: pointer;
  transition: background 0.3s, transform 0.2s;
}

.styled-button:hover {
  background-color: #c70039;
  transform: scale(1.1);
}
</style>
