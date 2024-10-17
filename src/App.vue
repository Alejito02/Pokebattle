<template>
  <div class="app">
    <div v-if="!isBattleStarted" class="home-page">
      <img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTxSBKFSUdL3VxbQXdSltIaE13yS4LT-df1Cw&s" class="Imagetittle">
      <h3>Ingresa tu nombre y elige el número de rondas para empezar la batalla</h3>
      <input
        v-model="playerName"
        placeholder="Ingresa tu nombre"
        @keyup.enter="startBattle"
      />
      <select v-model="totalRounds">
        <option disabled value="">Selecciona el número de rondas</option>
        <option v-for="round in [3, 5, 7]" :key="round" :value="round">
          {{ round }}
        </option>
      </select>
      <button @click="startBattle">Iniciar Batalla</button>
    </div>

    <div v-else class="battle-page">
      <h2>{{ playerName }} vs Bot</h2>
      <p>Ronda actual: {{ currentRound }} / {{ totalRounds }}</p>
      <div class="scoreboard">
        <p>{{ playerName }}: {{ playerWins }} victorias</p>
        <p>Bot: {{ botWins }} victorias</p>
      </div>
      <div class="teams">
        <div class="team">
          <h3>{{ playerName }}</h3>
          <div
            v-for="(pokemon, index) in playerTeam"
            :key="index"
            :class="['pokemon-card', { winner: isWinner('player') }]"
          >
            <img :src="pokemon.image" :alt="pokemon.name" />
            <h4>{{ pokemon.name }}</h4>
            <p>HP: {{ pokemon.stats.hp }}</p>
            <p>Ataque: {{ pokemon.stats.attack }}</p>
            <p>Defensa: {{ pokemon.stats.defense }}</p>
          </div>
        </div>
        <div class="team">
          <h3>Bot</h3>
          <div
            v-for="(pokemon, index) in botTeam"
            :key="index"
            :class="['pokemon-card', { winner: isWinner('bot') }]"
          >
            <img :src="pokemon.image" :alt="pokemon.name" />
            <h4>{{ pokemon.name }}</h4>
            <p>HP: {{ pokemon.stats.hp }}</p>
            <p>Ataque: {{ pokemon.stats.attack }}</p>
            <p>Defensa: {{ pokemon.stats.defense }}</p>
          </div>
        </div>
      </div>
      <select v-model="selectedStat">
        <option disabled value="">Selecciona una característica</option>
        <option value="hp">HP</option>
        <option value="attack">Ataque</option>
        <option value="defense">Defensa</option>
      </select>
      <button @click="playRound">Jugar Ronda</button>
      <div v-if="roundResult">
        <p>{{ roundResult }}</p>
      </div>
    </div>

    <div v-if="showModal" class="modal" @click="closeModal">
      <div class="modal-content" @click.stop>
        <h2>¡Juego Terminado!</h2>
        <p>Ganador: <strong>{{ winnerName }}</strong></p>
        <button @click="closeModal">Cerrar</button>
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
      lastRoundWinner: null,
      showModal: false,
      winnerName: '',
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
        stats: {
          hp: response.data.stats.find((s) => s.stat.name === 'hp').base_stat,
          attack: response.data.stats.find((s) => s.stat.name === 'attack').base_stat,
          defense: response.data.stats.find((s) => s.stat.name === 'defense').base_stat,
        },
      }));
      this.playerTeam = pokemons.slice(0, 3);
      this.botTeam = pokemons.slice(3, 6);
    },
    async regenerateTeam(team) {
      const pokemonIds = Array.from({ length: 3 }, () =>
        Math.floor(Math.random() * 898) + 1
      );
      const promises = pokemonIds.map((id) =>
        axios.get(`https://pokeapi.co/api/v2/pokemon/${id}`)
      );
      const responses = await Promise.all(promises);
      const newPokemons = responses.map((response) => ({
        name: response.data.name,
        image: response.data.sprites.front_default,
        stats: {
          hp: response.data.stats.find((s) => s.stat.name === 'hp').base_stat,
          attack: response.data.stats.find((s) => s.stat.name === 'attack').base_stat,
          defense: response.data.stats.find((s) => s.stat.name === 'defense').base_stat,
        },
      }));

      if (team === 'player') {
        this.playerTeam = newPokemons;
      } else {
        this.botTeam = newPokemons;
      }
    },
    async playRound() {
      if (!this.selectedStat) return;

      const playerStat = this.playerTeam[0].stats[this.selectedStat];
      const botStat = this.botTeam[0].stats[this.selectedStat];

      if (playerStat > botStat) {
        this.roundResult = `${this.playerName} gana la ronda!`;
        this.lastRoundWinner = 'player';
        this.playerWins++;
        await this.regenerateTeam('bot'); 
      } else if (playerStat < botStat) {
        this.roundResult = 'Bot gana la ronda!';
        this.lastRoundWinner = 'bot';
        this.botWins++;
        await this.regenerateTeam('player'); 
      } else {
        this.roundResult = 'Empate!';
        this.lastRoundWinner = 'draw';
      }

      if (this.currentRound < this.totalRounds) {
        this.currentRound++;
      } else {
        this.calculateWinner();
      }

      this.selectedStat = '';
    },
    isWinner(team) {
      return this.lastRoundWinner === team;
    },
    calculateWinner() {
      if (this.playerWins > this.botWins) {
        this.winnerName = this.playerName;
      } else if (this.botWins > this.playerWins) {
        this.winnerName = 'Bot';
      } else {
        this.winnerName = 'Empate';
      }

      this.showModal = true;
    },
    closeModal() {
      this.showModal = false;
      this.resetGame();
    },
    resetGame() {
      this.playerName = '';
      this.totalRounds = '';
      this.currentRound = 1;
      this.playerTeam = [];
      this.botTeam = [];
      this.selectedStat = '';
      this.roundResult = '';
      this.lastRoundWinner = null;
      this.playerWins = 0;
      this.botWins = 0;
      this.isBattleStarted = false;
    },
  },
};
</script>



<style scoped>
.app {
  width: 100%;
  text-align: center;
  background-image: url("https://media1.tenor.com/m/fysPvalDmXcAAAAC/ataque-pokemon-batalla-pokemon.gif");
  background-size: cover;
  height: 100vh;
  font-family: 'Arial', sans-serif;
  color: black;
  background-color: black;
}

.home-page {
  background: rgba(255, 255, 255, 0.9);
  border-radius: 10px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.2);
  display: flex;
  flex-direction: column;
  height: 70%;
  width: 80%;
  justify-content: center;
  margin-left: 10%;
}

.Imagetittle{
  width: 50%;
  height: 40%;
  margin-left: 25%;
}

input, select {
  padding: 10px;
  margin-right: 30%;
  margin-left: 30%;
  border: 2px solid #42b983;
  border-radius: 5px;
}

button {
  padding: 10px 20px;
  background-color: #42b983;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s;
  margin-right: 30%;
  margin-left: 30%;
}

button:hover {
  background-color: #36a76a;
}

.battle-page {
  margin-top: 20px;
  background: rgba(255, 255, 255, 0.555);
  border-radius: 10px;
  padding: 20px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.2);
}

.teams {
  display: flex;
  justify-content: space-around;
  margin-top: 20px;
}

.team {
  width: 30%;
  background-color: #1ed3f3a6;
  border-radius: 10px;
  padding: 10px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
}

.pokemon-card {
  border: 1px solid #ddd;
  padding: 10px;
  margin-bottom: 10px;
  border-radius: 10px;
  transition: all 0.3s ease;
  background-color: #ffffff;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}

.pokemon-card.winner {
  animation: pulse 0.5s infinite alternate;
  background-color: #e0f7e7;
  border-color: #42b983;
}

.pokemon-card img {
  width: 100px;
  height: 100px;
}

select, button {
  margin-top: 10px;
  padding: 10px;
  border-radius: 5px;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.8);
  display: flex;
  justify-content: center;
  align-items: center;
  animation: fadeIn 0.5s;
}

.modal-content {
  background-image: url(https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQrRMSXVmMPziC-OM1Xkh8lIOKCjjksT0M_3Q&s);
  padding: 10%;
  border-radius: 10px;
  text-align: center;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
  animation: slideIn 0.5s;
  background-size: 100%;
  background-repeat: no-repeat;
  display: flex;
  flex-direction: column;
  align-items: center;
}

@keyframes pulse {
  from {
    transform: scale(1);
    box-shadow: 0 0 10px rgba(66, 185, 131, 0.5);
  }
  to {
    transform: scale(1.05);
    box-shadow: 0 0 20px rgba(66, 185, 131, 0.8);
  }
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

@keyframes slideIn {
  from {
    transform: translateY(-50px);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
  }
}
</style>
