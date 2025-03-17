<template>
  <div class="app">
    <div v-if="!isBattleStarted" class="home-page">
      <img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTxSBKFSUdL3VxbQXdSltIaE13yS4LT-df1Cw&s" class="Imagetittle">
      <h3>Ingresa tu nombre y elige el número de rondas para empezar la batalla</h3>
      <div class="input-container">
        <input v-model="playerName" placeholder="Ingresa tu nombre" class="styled-input" />
        <select v-model="totalRounds" class="styled-select">
          <option disabled value="">Selecciona el número de rondas</option>
          <option v-for="round in [3, 5, 7]" :key="round" :value="round">{{ round }}</option>
        </select>
        <button @click="startBattle" class="styled-button">Iniciar Batalla</button>
      </div>
    </div>

    <div v-else class="battle-page">
      <h2>{{ playerName }} vs Bot</h2>
      <p>Ronda actual: {{ currentRound }} / {{ totalRounds }}</p>
      <div class="battle-field">
        <div class="player-side">
          <div v-if="playerTeam.length" class="pokemon-container">
            <div class="info-box player-info">
              <p>{{ playerTeam[0].name.toUpperCase() }}</p>
              <p>HP: {{ playerTeam[0].stats.hp }}</p>
            </div>
            <img v-if="playerTeam[0].backImage" :src="playerTeam[0].backImage" :alt="playerTeam[0].name" class="pokemon-back" />
          </div>
        </div>
        <div class="bot-side">
          <div v-if="botTeam.length" class="pokemon-container">
            <img :src="botTeam[0].image" :alt="botTeam[0].name" class="pokemon-front" />
            <div class="info-box bot-info">
              <p>{{ botTeam[0].name.toUpperCase() }}</p>
              <p>HP: {{ botTeam[0].stats.hp }}</p>
            </div>
          </div>
        </div>
      </div>
      <div class="input-container" v-if="currentRound <= totalRounds">
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

    <div v-if="showWinnerModal" class="winner-modal">
      <div class="modal-content">
        <img src="https://cdn-icons-png.flaticon.com/512/1828/1828884.png" alt="Trophy" class="trophy" />
        <h2>¡Ganador!</h2>
        <p>{{ winner }}</p>
        <button @click="resetGame" class="styled-button">Volver a Inicio</button>
      </div>
    </div>
  </div>
</template>

<script>
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
      showWinnerModal: false,
      winner: ''
    };
  },
  methods: {
    startBattle() {
      if (this.playerName.trim() && this.totalRounds) {
        this.isBattleStarted = true;
        this.currentRound = 1;
        this.playerWins = 0;
        this.botWins = 0;
        this.fetchPokemon();
      }
    },
    resetGame() {
      this.isBattleStarted = false;
      this.playerName = '';
      this.totalRounds = '';
      this.currentRound = 1;
      this.playerWins = 0;
      this.botWins = 0;
      this.playerTeam = [];
      this.botTeam = [];
      this.selectedStat = '';
      this.roundResult = '';
      this.showWinnerModal = false;
    },
    async fetchPokemon() {
      try {
        const pokemonIds = Array.from({ length: 6 }, () =>
          Math.floor(Math.random() * 898) + 1
        );
        const pokemons = await Promise.all(
          pokemonIds.map(async (id) => {
            const response = await fetch(`https://pokeapi.co/api/v2/pokemon/${id}`);
            const data = await response.json();
            return {
              name: data.name,
              image: data.sprites.front_default,
              backImage: data.sprites.back_default || null,
              stats: {
                hp: data.stats.find((s) => s.stat.name === 'hp').base_stat,
                attack: data.stats.find((s) => s.stat.name === 'attack').base_stat,
                defense: data.stats.find((s) => s.stat.name === 'defense').base_stat,
              },
            };
          })
        );
        this.playerTeam = pokemons.slice(0, 3);
        this.botTeam = pokemons.slice(3, 6);
      } catch (error) {
        console.error("Error al obtener los Pokémon:", error);
      }
    },
    playRound() {
      if (!this.selectedStat) return;
      const playerStat = this.playerTeam[0].stats[this.selectedStat];
      const botStat = this.botTeam[0].stats[this.selectedStat];
      if (playerStat > botStat) {
        this.roundResult = `¡Ganaste la ronda! (${playerStat} vs ${botStat})`;
        this.playerWins++;
      } else if (botStat > playerStat) {
        this.roundResult = `El Bot gana la ronda. (${botStat} vs ${playerStat})`;
        this.botWins++;
      } else {
        this.roundResult = "Empate en esta ronda.";
      }
      this.currentRound++;
      this.selectedStat = '';
      if (this.currentRound > this.totalRounds) {
        setTimeout(() => {
          this.winner = this.playerWins > this.botWins ? this.playerName : 'El Bot';
          this.showWinnerModal = true;
        }, 1000);
      }
    }
  }
};
</script>

<style scoped>
.winner-card {
  background: rgba(0, 0, 0, 0.7);
  padding: 20px;
  border-radius: 10px;
  text-align: center;
}
.trophy-container {
  display: flex;
  justify-content: center;
  margin: 10px 0;
}
.trophy {
  width: 100px;
}
</style>


<style scoped>
.app {
  width: 100%;
  text-align: center;
  background-image: url("https://fiverr-res.cloudinary.com/images/t_main1,q_auto,f_auto,q_auto,f_auto/v1/attachments/delivery/asset/0032398f86ea753194c5eeba97eccda2-1627249600/ExportBackgroundnomoveclound/draw-a-pixel-pokemon-battle-background.gif");
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
  width: 160px;
  height: 160px;
}

.pokemon-front{
  width: 160px;
  height: 160px;
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

.pokemon-back {
  transform: scaleX(1);
}

.info-box {
  background-image: url(https://e1.pxfuel.com/desktop-wallpaper/654/268/desktop-wallpaper-i-edited-an-of-the-pokedex-to-work-nicely-as-an-iphone-6s-lock-screen-thesilphroad-pokedex-thumbnail.jpg);
  background-size: cover;
  color: rgb(255, 255, 255);
  padding: 10px;
  text-align: center;
  margin-top: 10px;
}

.battle-field {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
  max-width: 800px;
  margin: auto;
  position: relative;
}

.pokemon-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  margin: 20px;
}

.info-box {
  padding: 10px;
  text-align: center;
  margin-top: 10px;
  width: 150px;
}

</style>
