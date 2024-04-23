<template>
  <v-app>
    <v-container>
      <v-row class="title" style="text-align:center;">
        <v-col cols="12">
          <h1 style="font-weight: 300; font-size: 4rem">Pokedex</h1> 
        </v-col>
      </v-row>
      <v-container class="pokemon-container">
        <v-text-field v-model="search" label="Pesquisar" placeholder="Pesquisar Pokémon" solo></v-text-field>
        <v-select v-model="searchType" :items="searchOptions" label="Tipo de Pesquisa" style="width: 10%;"></v-select>
        <v-row class="row">
          <v-col cols="3" v-for="pokemon in filtered_pokemon" :key="pokemon.name">
            <v-card @click="show_pokemon(pokemon)" class="pokemon-card" :flat="true">
              <v-container>
                <v-row class="mx-0 d-flex justify-center">
                  <div class="pokemon-image-container">
                    <img :src="getImageUrl(pokemon)" :alt="pokemon.name" class="pokemon-image">
                  </div>
                </v-row>
                <h2 class="text-center">{{ get_name(pokemon) }}</h2>
                <p class="text-center">ID: {{ pokemon.id }}</p>
              </v-container>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-container>
    <v-dialog v-model="dialog" width="500">
      <template v-slot:activator="{ on }">
        <v-btn color="red lighten-2" dark v-on="on">
        </v-btn>
      </template>
      <v-card v-if="selected_pokemon">
        <v-card-text>
          <v-container class="detail">
            <img :src="getImageUrl(selected_pokemon)" :alt="selected_pokemon.name" class="pokemon-image-id">
            <h2 class="mx-0 d-flex justify-center">{{ selected_pokemon.name }}</h2>
            <div class="property">
              <div class="left">Base Experience</div>
              <div class="right">{{ selected_pokemon.base_experience }} XP</div>
            </div>
            <div class="property">
              <div class="left">ID</div>
              <div class="right">{{ selected_pokemon.id }}</div>
            </div>
            <div class="property">
              <div class="left">Altura</div>
              <div class="right">{{ selected_pokemon.height / 10 }} m</div>
            </div>
            <div class="property">
              <div class="left">Peso</div>
              <div class="right">{{ selected_pokemon.weight }} kg</div>
            </div>
            <h3>Pokemon Tipo</h3>
            <div class="types">
              <div class="type" v-for="(type, index) in selected_pokemon.types" :key="index">
                {{ type.type.name }}
              </div>
            </div>
            <h3>Habilidade</h3>
            <div class="habilities">
              <div class="hability" v-for="(ability, index) in selected_pokemon.abilities" :key="index">
                {{ ability.ability.name }}
              </div>
            </div>
            <h3>Evoluções</h3>
            <div>
              <v-row class="mx-0 d-flex align-center">
                <template v-for="(evolution_detail, index) in evolutions">
                  <v-col cols="12" md="3" :key="`evolution-${index}`">
                    <v-card v-if="evolution_detail.imageUrl" :flat="true" class="evolution-card">
                      <v-container>
                        <img :src="evolution_detail.imageUrl" :alt="evolution_detail.name" class="evolution-card">
                        <h2 class="mx-0 d-flex justify-center">{{ evolution_detail.name }}</h2>
                      </v-container>
                    </v-card>
                    <v-col v-else>
                      <h3 class="text-center">{{ evolution_detail.name }}</h3>
                    </v-col>
                  </v-col>
                </template>
              </v-row>
            </div>
            <div>
    <div v-if="gamesIndices.length > 0">
      <div class="card">
        <div class="card-header">
          <h2>Game Indices</h2>
        </div>
        <div class="card-body">
          <table class="table">
            <thead>
              <tr>
                <th>Versão</th>
                <th>Índice do Jogo</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(gameIndex, index) in gamesIndices" :key="index">
                <td>{{ gameIndex.versionName }}</td>
                <td>{{ gameIndex.gameIndex }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
    <div v-else>
      <p>Não há informações de jogo disponíveis.</p>
    </div>
  </div>
          </v-container>
        </v-card-text>
      </v-card>
    </v-dialog>
  </v-app>
</template>

<script>
import axios from "axios";
export default {
  name: 'App',
  components: {},
  data() {
    return {
      pokemons: [],
      search: "",
      searchType: "nome",
      searchOptions: ["nome", "id", "tipo", "espécie"],
      dialog: false,
      selected_pokemon: null,
      evolution: null,
      evolutions: [], 
      gamesIndices: []
    };
  },
  mounted() {
    axios.get("https://pokeapi.co/api/v2/pokemon?limit=493").then((response) => {
      this.pokemons = response.data.results.map(pokemon => ({
        ...pokemon,
        id: this.getId(pokemon)
      }));
      if (this.pokemons.length > 0) {
        axios.get(this.pokemons[0].url).then(response => {
          this.selected_pokemon = response.data;
        });
      }
    });
  },
  methods: {
    getId(pokemon) {
      return pokemon.url.split("/")[6];
    },
    getImageUrl(pokemon) {
      return `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${pokemon.id}.png`;
    },
    get_name(pokemon) {
      return pokemon.name.charAt(0).toUpperCase() + pokemon.name.slice(1);
    },
    show_pokemon(pokemon) {
      axios.get(pokemon.url).then((response) => {
        this.selected_pokemon = response.data;
        this.fetch_evolution();
        this.fetch_games_indices(); 
        this.dialog = true;
      });
    },
    fetch_games_indices() {
      axios.get(`https://pokeapi.co/api/v2/pokemon/${this.selected_pokemon.id}`)
  .then(response => {
    const pokemonData = response.data;
    if (pokemonData.game_indices && pokemonData.game_indices.length > 0) {
      const promises = pokemonData.game_indices.map(indexData => {
        return axios.get(indexData.version.url);
      });

      Promise.all(promises)
        .then(responses => {
          const gamesIndices = responses.map((response, index) => {
            const versionName = response.data.name;
            const gameIndex = pokemonData.game_indices[index].game_index;
            return {
              versionName: versionName,
              gameIndex: gameIndex
            };
          });
          this.gamesIndices = gamesIndices; // Retorna os dados do jogo para o componente
        })
        .catch(error => {
          console.error('Erro ao buscar dados das versões:', error);
        });
    } else {
      this.gamesIndices = []; // Se não houver informações de game_indices, define gamesIndices como vazio
    }
  })
  .catch(error => {
    console.error('Erro ao buscar dados do Pokémon:', error);
  });},
    fetch_evolution() {
      axios.get(`https://pokeapi.co/api/v2/pokemon/${this.selected_pokemon.id}`).then((response) => { 
        const speciesName = response.data.species.name; 
        axios.get(`https://pokeapi.co/api/v2/pokemon-species/${speciesName}`).then((response) => {
          if (response.data.evolution_chain) {
            const evolutionChainUrl = response.data.evolution_chain.url;
            axios.get(evolutionChainUrl).then((response) => {
              const evolutionChain = response.data.chain;
              this.evolution = evolutionChain;
              this.updateEvolutions(); 
            });
          } else { 
            this.evolution = null;
            this.evolutions = []; 
          }
        }).catch(error => {
          console.error('Erro ao buscar dados de evolução:', error);
        });
      });
    },
    updateEvolutions() {
      let chain = [];
      let evolution = this.evolution;
      
      const getPokemonImageUrlById = (id) => {
        return `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${id}.png`;
      };

      while (evolution && evolution.species) {
        if (evolution.species) {
          chain.push({
            name: evolution.species.name,
            imageUrl: getPokemonImageUrlById(evolution.species.url.split("/")[6])
          });
        }
        if (evolution.evolves_to && evolution.evolves_to.length > 0) {
          evolution = evolution.evolves_to[0];
        } else {
          break;
        }
      }

      this.evolutions = chain;
    },
  },
  computed: {
    filtered_pokemon() {
      if (!this.search) return this.pokemons;
      switch (this.searchType) {
        case "id":
          return this.pokemons.filter(pokemon => pokemon.id.toString().includes(this.search));
          case "tipo":
            return this.pokemons.filter(pokemon => 
              pokemon.types && pokemon.types.some(type => type.type.name.toLowerCase().includes(this.search.toLowerCase()))
            );
        case "espécie":
          return this.pokemons.filter(pokemon => 
            pokemon.species.toLowerCase().includes(this.search.toLowerCase())
          );
        case "nome":
        default:
          return this.pokemons.filter(pokemon => pokemon.name.toLowerCase().includes(this.search.toLowerCase()));
      }
    }
  }
};

</script>

<style>
#app {
  background: linear-gradient(
      to bottom right,
      #B296FF,
      #C1D2DC
    )
    no-repeat center center fixed !important;
}
.title{
   text-align: center;
   margin: 2% 70% 0 30%;
  color: #efefef;
  font-family: 'Acme', arial;
  font-size: 10rem;
  font-weight: inherit;
}

.pokemon-container {
  margin-top: 4%;
  width: 80%;
  
}
.pokemon-card {
  background-color: black;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease-in-out;
  margin-top: 15%;
  text-align: center;
}
.pokemon-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.1);
}
.pokemon-image-container {
  margin-top: -50px;
  margin-bottom: 20px;
  text-align: center;
}
.pokemon-image {
  width: 150px;
  height: 150px;
  border-radius: 50%;
  border: 5px solid #fff;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}
.text-center {
  text-align: center;  
}
.pokemon-image-id {
  width: 150px;
  height: 150px;
  border-radius: 30%;
  border: 5px solid #B296FF;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  display: block; 
  margin:  0 auto; 
 
}
.property {
   margin-top: 5%;
   width: 90%;
   max-width: 400px;
  border-bottom: 1px solid #ccc;
  margin-bottom: 40px;

  .left { float: left; }
  .right { float: right; }
}

.types, .habilities {
          display: flex;
          justify-content: flex-start;
          flex-wrap: wrap;
          width: 90%;
          max-width: 400px;
}          

.type, .hability {
            margin: 5px 10px 10px 0;
            padding: 5px 10px;
            border-radius: 20px;
            color: #fff;
            font-size: 1rem;
            letter-spacing: 2px;
            text-transform: capitalize;
            word-wrap: none;
            word-break: keep-all;
}

.type { background-color:#B296FF; }
.hability { background-color: #C1D2DC; }

.evolution-card{
  margin-right: 20%;
}
.table {
  width: 100%;
  border-collapse: collapse;
  border-radius: 5px;
  overflow: hidden;
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.1);
  margin-top: 20px;
}

.table th {
  background-color: #f2f2f2;
  color: #333;
  font-weight: bold;
  padding: 10px;
  text-align: left;
  border-bottom: 1px solid #ddd;
}

.table td {
  padding: 10px;
  border-bottom: 1px solid #ddd;
}

.table td:first-child {
  font-weight: bold;
}

.table .highlight {
  background-color: #f5f5f5;
}

.table tr:nth-child(odd) {
  background-color: #fafafa;
}
</style>
