<template>
  <div :class="[darkMode ? 'text-white' : 'text-black', getBackgroundColor(), 'w-screen h-screen overflow-hidden']">
    <div class="flex w-full h-full relative">
      <!-- Buttons for dark/light mode and sound in the top right corner -->
      <div class="absolute top-4 right-4 space-x-4">
        <button class="song-button bg-gray-700 text-white px-4 py-2 rounded" @click="toggleMusic">
          <img :src="musicPlaying ? '/images/soundon.png' : '/images/soundoff.png'" alt="Music Control" class="w-6 h-6" />
        </button>
        <button @click="toggleDarkMode" class="song-button bg-gray-700 text-white px-4 py-2 rounded">
          <img :src="darkMode ? '/images/dark-mode.png' : '/images/light-mode.png'" alt="Light Control" class="w-6 h-6" />
        </button>
      </div>
      <div v-if="pokemon" :class="[darkMode ? 'bg-opacity-70' : 'bg-opacity-100', getBackgroundColor(), 'w-1/2 flex flex-col items-center justify-center']">
        <div :class="[darkMode ? 'poke-card-dark' : 'poke-card-light', 'poke-card p-4 text-center relative']">
          <img class="poke-logo mt-4 mx-auto w-68" src="/images/pokelogo5.png" alt="Poke logo" />
          <h2 class="poke-name mt-0 mb-5 text-4xl font-bold">{{ capitalizedPokemonName }}</h2> <!-- Pokemon name with first letter uppercased -->
          <div class="pokemon-image-wrapper relative flex justify-center">
            <!-- 1/50 chance of getting a shiny pokémon, if its the case shiny version will be displayed -->
            <img class="pokemon-image" :src="isShiny ? pokemon.sprites.front_shiny : pokemon.sprites.front_default" :alt="pokemon.name" />
            <!-- Shiny icon at the right top corner of the img box when pokémon is shiny -->
            <img v-if="isShiny" class="shiny-symbol absolute mr-60 top-4 right-0 w-8 h-8" src="/images/shiny-symbol.png" alt="Shiny Symbol" />
          </div>
          <!-- Button to match the type colour of the pokemon displayed -->
          <button :class="[getTypeColor(pokemon.types[0].type.name), 'random-button text-white px-4 py-2 mt-6 rounded']" @click="fetchRandomPokemon">New Pokémon!</button>
          <p class="text-sm mt-7 mb-8">Click to discover a random Pokémon,<br/> if you're lucky you may find a shiny one! </p>
          <audio id="myAudio" loop>
            <source src="/poke-soundtrack.mp3" type="audio/mp3"><!-- Pokémon retro music -->
          </audio>
        </div>
      </div>

      <div v-if="pokemon" :class="[darkMode ? 'bg-opacity-70' : 'bg-opacity-100', getBackgroundColor(), 'info-part w-1/2 flex flex-col items-center justify-center']"><!-- Dark and light mode adjustments -->
        <div class="right-side text-center mt-4">
          <ul class="list-none">
            <div class="types-box flex space-x-2 mb-4">
              <h3 class="text-lg mt-0 ml-8 font-bold">Types</h3>  
              <div class="types-info flex mr-8"> <!-- Displays the pokémon's type(s) -->
                <li v-for="type in pokemon.types" :key="type.type.name" :class="[getTypeColor(type.type.name)]" class="type-tag mr-8 px-2 py-1 rounded text-white capitalize">{{ type.type.name }}</li>
              </div> 
            </div>
          </ul>
          <ul class="stats grid grid-cols-2 column left-1 gap-0 p-0 my-2">
            <li v-for="stat  in pokemon.stats" :key="stat.stat.name" class="stats-info capitalize text-left">{{ stat.stat.name }}: {{ stat.base_stat }}</li>
            <li class="total-info font-bold col-span-2 text-left">Total: {{ totalStats }}</li>
          </ul>
          <div class="info-grid grid grid-cols-2 column left-1 gap-0 p-0 mt-4">
            <!-- Display pokemon information -->
            <p class="grid-info text-sm my-2">Region: {{ region }}</p>
            <p class="grid-info text-sm my-2">Height: {{ convertedHeight }} cm</p>
            <p class="grid-info text-sm my-2">Base Exp: {{ pokemon.base_experience }}</p>
            <p class="grid-info text-sm my-2">Weight: {{ convertedWeight }} kg</p>
          </div>
          <!-- Displays pokemon weaknesses -->
          <h3 class="text-lg font-bold mt-2">Weaknesses</h3>
          <div v-if="weaknesses.length > 0" class="weaknesses-list flex justify-center space-x-2 mt-2">
            <span v-for="(weakness, index) in weaknesses" :key="index" :class="[getTypeColor(weakness)]" class="weakness-tag px-2 py-1 rounded text-white">{{ weakness }}</span>
          </div>
          
          <!-- Displays pokemon evolution chain -->
          <h2 class="text-lg font-bold mt-2">Evolution Chain</h2>
          <div v-if="evolutionImages.length > 0" class="evolution-chain flex space-x-4 mt-2">
            <img v-for="(image, index) in evolutionImages" :key="index" :src="image" class="evolution-image w-20 h-20" />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'HomePage',
  data() {
    return {
      pokemon: null,
      loading: true,
      musicPlaying: false,
      darkMode: true,
      isShiny: false,
      evolutionImages: [],
      weaknesses: [],
      region: '',
      typeColors: {
        normal: 'bg-gray-400',
        fire: 'bg-red-500',
        water: 'bg-blue-500',
        grass: 'bg-green-500',
        electric: 'bg-yellow-500',
        ice: 'bg-blue-300',
        fighting: 'bg-red-700',
        poison: 'bg-purple-500',
        ground: 'bg-yellow-700',
        flying: 'bg-indigo-300',
        psychic: 'bg-pink-500',
        bug: 'bg-green-700',
        rock: 'bg-gray-700',
        ghost: 'bg-purple-700',
        dark: 'bg-gray-800',
        dragon: 'bg-indigo-700',
        steel: 'bg-gray-500',
        fairy: 'bg-pink-300',
      }
    };
  },
  async mounted() {
    await this.fetchRandomPokemon();
    this.startMusic();
  },
  computed: {
    capitalizedPokemonName() {
      if (!this.pokemon) return '';
      return this.pokemon.name.charAt(0).toUpperCase() + this.pokemon.name.slice(1); // Changing the first letter of the pokémon's name to uppercase letter
    },
    totalStats() {
      if (!this.pokemon) return 0;
      return this.pokemon.stats.reduce((total, stat) => total + stat.base_stat, 0);
    },
    convertedHeight() {
      if (!this.pokemon) return 0;
      return this.pokemon.height * 10; // Converts decimetres to centimetres
    },
    convertedWeight() {
      if (!this.pokemon) return 0;
      return this.pokemon.weight / 10; // Converts hectograms to kilograms
    }
  },
  methods: {
    async fetchRandomPokemon() {
      try {
        const randomId = Math.floor(Math.random() * 1302) + 1;
        const response = await axios.get(`https://pokeapi.co/api/v2/pokemon/${randomId}`);
        this.pokemon = response.data;
        this.isShiny = Math.random() < 0.02; // 1/50 chance of shiny Pokémon
        this.fetchEvolutionChain(this.pokemon.species.url);
        this.calculateWeaknesses(this.pokemon.types);
        this.fetchRegion(this.pokemon.species.url);
      } catch (error) {
        console.error(error);
      } finally {
        this.loading = false;
      }
    },
    async fetchEvolutionChain(speciesUrl) {
      try {
        const speciesResponse = await axios.get(speciesUrl);
        const evolutionChainUrl = speciesResponse.data.evolution_chain.url;
        const evolutionResponse = await axios.get(evolutionChainUrl);
        this.evolutionImages = this.extractEvolutionImages(evolutionResponse.data.chain);
      } catch (error) {
        console.error(error);
      }
    },
    async fetchRegion(speciesUrl) {
      try {
        const speciesResponse = await axios.get(speciesUrl);
        const generationResponse = await axios.get(speciesResponse.data.generation.url);
        this.region = generationResponse.data.main_region.name;
      } catch (error) {
        console.error(error);
      }
    },
    extractEvolutionImages(chain) {
      const images = [];
      let currentChain = chain;

      while (currentChain) {
        const speciesUrl = currentChain.species.url;
        const id = speciesUrl.split('/').slice(-2, -1)[0];
        images.push(`https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${id}.png`);
        currentChain = currentChain.evolves_to[0];
      }

      return images;
    },
    async calculateWeaknesses(types) {
      const weaknessesSet = new Set();
      try {
        for (const type of types) {
          const typeResponse = await axios.get(type.type.url);
          const doubleDamageFrom = typeResponse.data.damage_relations.double_damage_from;
          doubleDamageFrom.forEach((weakness) => {
            weaknessesSet.add(weakness.name);
          });
        }
        this.weaknesses = Array.from(weaknessesSet);
      } catch (error) {
        console.error(error);
      }
    },
    getPokemonHP() {
      if (!this.pokemon) return null;
      const hpStat = this.pokemon.stats.find(stat => stat.stat.name === 'hp');
      return hpStat ? hpStat.base_stat : 'N/A';
    },
    getBackgroundColor() {
      if (!this.pokemon || !this.pokemon.types || this.pokemon.types.length === 0) {
        return this.darkMode ? 'bg-gray-900' : 'bg-gray-100'; // Default background color
      }
      const type = this.pokemon.types[0].type.name; // Get the primary type
      const typeColor = this.typeColors[type] || 'bg-gray-100'; // Default to gray if type is not found
      return this.darkMode ? `${typeColor} bg-opacity-70` : `${typeColor} bg-opacity-100`;
    },
    getTypeColor(type) {
      return this.typeColors[type] || 'bg-gray-100'; // Default to gray if type is not found
    },
    startMusic() {
      const audioElement = document.getElementById('myAudio');
      if (audioElement) {
        audioElement.play().catch(error => {
          console.error('Error playing audio:', error);
        });
        this.musicPlaying = true;
      }
    },
    toggleMusic() {
      const audioElement = document.getElementById('myAudio');
      if (audioElement) {
        if (this.musicPlaying) {
          audioElement.pause();
        } else {
          audioElement.play().catch(error => {
            console.error('Error playing audio:', error);
          });
        }
        this.musicPlaying = !this.musicPlaying;
      }
    },
    toggleDarkMode() {
      this.darkMode = !this.darkMode;
    },
    scrollToContent(selector) {
      const content = document.querySelector(selector);
      if (content) {
        content.scrollIntoView({ behavior: 'smooth' });
      }
    }
  }
};
</script>

<style>
.song-button{
  border: 1px solid transparent;
}
.song-button:hover{
  border: 1px solid whitesmoke;
}
.song-button:focus{
  border: 1px solid transparent;
}
.poke-logo {
  width: 300px;
}
.text-lg {
  margin-top: 40px;
}
.poke-card {
  width: 100%;
  height: 100vh;
}
.poke-card-dark {
  background-color: rgba(42, 48, 56, 0.603);
}
.poke-card-light {
  background-color: rgba(228, 228, 228, 0.689);
}
.pokemon-image {
  height: 270px;
  width: 270px;
  background-color: rgba(167, 167, 167, 0.892);
  margin-bottom: 20px;
  border-radius: 12px;
  padding: 5px;
}
.random-button {
  font-family: "Nunito Sans", sans-serif;
  font-optical-sizing: auto;
  font-weight: 800;
  font-style: normal;
  font-variation-settings: "wdth" 100, "YTLC" 500;
  border: 2px solid transparent;
}
.random-button:hover {
  color: black;
  border: 2px solid whitesmoke;
}
.random-button::selection {
  border: 2px solid transparent;
}
.info-part {
  background-color: rgba(25, 31, 40, 0.569);
}
.flex {
  font-family: "Nunito Sans", sans-serif;
  font-optical-sizing: auto;
  font-weight: 800;
  font-style: normal;
  font-variation-settings: "wdth" 100, "YTLC" 500;
}
.types-box{
  background-color: rgba(156, 156, 156, 0.745);
  height: 60px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  border-radius: 12px;
}
.text-center {
  line-height: 2;
  padding: 40px;
}
.evolution-image {
  width: 70px;
  height: 70px;
}
.evolution-chain {
  background-color: rgba(156, 156, 156, 0.745);
  height: 80px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 12px;
}
.weaknesses-list{
  background-color: rgba(156, 156, 156, 0.745);
  height: 80px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 12px;
}
.weakness-tag {
  font-size: 14px;
}
.type-tag {
  font-size: 14px;
}
.stats {
  align-items: start;
  background-color: rgba(156, 156, 156, 0.745);
  height: 200px;
  padding-left: 20px;
  padding-right: 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  border-radius: 12px;
  line-height: 1;
}
.stats-info{
  margin-left: 30px;
}
.total-info{
  margin-left: 30px;
}
.bg-gray-900 {
  background-color: #1a202c;
}
.bg-gray-100 {
  background-color: #f7fafc;
}
.bg-gray-800 {
  background-color: #2d3748;
}
.bg-gray-200 {
  background-color: #edf2f7;
}
.bg-gray-700 {
  background-color: #4a5568;
}
.bg-gray-300 {
  background-color: #e2e8f0;
}
.text-white {
  color: #ffffff;
}
.text-black {
  color: #000000;
}
.text-left {
  text-align: left;
}
.info-grid{
  background-color: rgba(156, 156, 156, 0.745);
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: left;
  border-radius: 12px;
}
.grid-info{
  margin-left: 30px;
}
.right-side{
  width: 500px;
}
</style>
