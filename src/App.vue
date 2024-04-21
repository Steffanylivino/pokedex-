<template>
  <v-app>
    <v-container>
                <v-row class="title" style="text-align:center; ">
            <v-col cols="6">
              <h1 style="font-weight: 300; font-size: 4rem">Pokedex</h1> 
          </v-col>
      </v-row>
      <v-container class="pokemon-container">
        <v-text-field v-model="search" label="Pesquisar" placeholder="Pesquisar Pokémon" solo></v-text-field>
        <v-select v-model="searchType" :items="searchOptions" label="Tipo de Pesquisa" style="width: 10%;"></v-select>
        <v-row class="row">
          <v-col cols="3" v-for="pokemon in filtered_pokemon" :key="pokemon.name">
            <v-card @click="show_pokemon(pokemon)" class="pokemon-card" :flat ="true">
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
          Click Me
        </v-btn>
      </template>
      <v-card>
        <v-card-text v-if="selected_pokemon">
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
              <div class="left">Height</div>
              <div class="right">{{ selected_pokemon.height / 10 }} m</div>
            </div>
            <div class="property">
              <div class="left">Peso</div>
              <div class="right">{{ selected_pokemon.weight }} kg</div>
            </div>
            <h3>Pokemon Tipos</h3>
            <div class="types">
              <div class="type" v-for="(value, index) in selected_pokemon.types" :key="'value'+index">
                {{ value.type.name }}
              </div>
            </div>
            <h3>Habilidades</h3>
            <div class="abilities">
              <div class="ability" v-for="(value, index) in selected_pokemon.abilities" :key="'value'+index">
                {{ value.ability.name }}
              </div>
            </div>
            <h3>Evolução</h3>
            <div>
              <v-row class="mx-0 d-flex align-center">
                <template v-for="(evolution_detail, index) in evolutions">
                  <v-col cols="12" md="3" :key="`evolution-${index}`">
                    <v-card v-if="evolution_detail.imageUrl" :flat ="true" class="evolution-card">
                      <v-container>
                        <img :src="evolution_detail.imageUrl" :alt="evolution_detail.name"  class="evolution-card">
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
            <h3>Indices de Jogos</h3>
            <div class="game-indices">
    
   
    <div class="ability" v-for="(gameIndex, index) in games_indices" :key="'gameIndex'+index">
          
          
    <div>Índice do Jogo: {{ gameIndex.gameIndex }}</div>
          
        
    <div>Nome do Movimento: {{ gameIndex.move.name }}</div>
          <div>URL do Movimento: {{ gameIndex.move.url }}</div>
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
      searchOptions: ["nome", "id","tipo","especie"],
      dialog: false,
      selected_pokemon: null,
      evolution: null,
      evolutions: [], // Adiciona a propriedade evolutions no objeto de dados
      games_indices: []
    };
  },
  mounted() {
    axios.get("https://pokeapi.co/api/v2/pokemon?limit=493").then((response) => {
      this.pokemons = response.data.results.map(pokemon => ({
        ...pokemon,
        id: this.getId(pokemon)
      }));
      
      // Verifica se há pelo menos um Pokémon antes de fazer a solicitação
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
        // Obtém as informações de evolução e games_indices
        this.fetch_evolution();
        this.fetch_games_indices(); 
        this.dialog = true;
      });
    },
    fetch_games_indices() {
      axios.get(this.selected_pokemon.species.url).then(response => {
        const speciesData = response.data;
        if (speciesData.game_indices) {
          const promises = speciesData.game_indices.map(gameIndex => {
            return axios.get(gameIndex.version.url);
          });

          Promise.all(promises).then(responses => {
            this.games_indices = responses.map(response => {
              return {
                versionName: response.data.name,
                gameIndex: speciesData.game_indices.find(gameIndex => gameIndex.version.url === response.config.url).game_index,
                move: {
                  name: speciesData.move.name,
                  url: speciesData.move.url
                }
              };
            });
          });
        } else {
          this.games_indices = [];
        }
      });
    },
    fetch_evolution() {
      axios.get(`https://pokeapi.co/api/v2/pokemon/${this.selected_pokemon.id}`).then((response) => { 
        const speciesName = response.data.species.name; 
        axios.get(`https://pokeapi.co/api/v2/pokemon-species/${speciesName}`).then((response) => {
          if (response.data.evolution_chain) {
            const evolutionChainUrl = response.data.evolution_chain.url;
            axios.get(evolutionChainUrl).then((response) => {
              const evolutionChain = response.data.chain;
              this.evolution = evolutionChain;
              this.updateEvolutions(); // Atualiza as evoluções após obter os dados
            });
          } else {
            // Pokémon não tem evolução
            this.evolution = null;
            this.evolutions = []; // Limpa as evoluções
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
          // Adiciona o Pokémon da evolução com sua imagem usando o ID
          chain.push({
            name: evolution.species.name,
            imageUrl: getPokemonImageUrlById(evolution.species.url.split("/")[6])
          });
        }
        // Verifica se há evoluções adicionais
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
          return this.pokemons.filter(pokemon => pokemon.type.toLowerCase().includes(this.search.toLowerCase()));
        case "espécie":
          return this.pokemons.filter(pokemon => pokemon.species.toLowerCase().includes(this.search.toLowerCase()));
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

.types, .abilities {
          display: flex;
          justify-content: flex-start;
          flex-wrap: wrap;
          width: 90%;
          max-width: 400px;
}          

.type, .ability {
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
.ability { background-color: #C1D2DC; }

.evolution-card{
  margin-right: 20%;
}
</style>
