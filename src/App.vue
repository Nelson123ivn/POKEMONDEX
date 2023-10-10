<template>
  <div class="principal">
    <div class="diseño">
      <div class="arriba"></div>
      <div class="recto">
        <div class="foto">
          <select class="form-select" aria-label="Default select example" v-model="tipoSeleccionado" placeholder="tipos" @change="filtrarpokemon()" style="text-align: center;">
            <option value="Todos">Todos</option>
            <option v-for="(color, tipo) in colorestipo" :key="color" :value="tipo" >
              {{ tipo }}
            </option>
          </select>
        </div>
        <div class="buscar">
          <input type="text" v-model="nombrepokemon">
          <button @click="filtrarPokemonPorNombre">BUSCAR</button>
        </div>
      </div>
      <div class="bola">
        <div class="bolapequeña"></div>
      </div>
      <div class="abajo"></div>
    </div>
    <div class="cartas">    
      <div v-for="(pokemon, index) in filtropokemones" :key="index" class="card" v-if="activarfiltro == true && filtronombre == false">
        <button  class="card-content" @click="abrirDetalle(index)">
          <div class="card-number">#{{ pokemon.numero }}</div>
          <div class="card-name">{{ pokemon.nombre }}</div>
          <img :src="pokemon.img" :alt="'Imagen de ' + pokemon.nombre" class="pokemon-image" />
          <div style="display: flex; justify-content: center; height: 10%; gap: 10%;">
            <div v-for="(tipo, i) in pokemon.tipos" :key="i" :style="'background-color: ' + colorestipo[tipo]" class="pokemon-type">
              <p>{{ tipo }}</p>
            </div>
          </div>
        </button>
      </div>
      <div v-for="(pokemon, index) in pokemons" :key="index" class="card" v-if="activarfiltro == false && filtronombre == false">
        <button  class="card-content" @click="abrirDetalle(index)">
          <div class="card-number">#{{ pokemon.numero }}</div>
          <div class="card-name">{{ pokemon.nombre }}</div>
          <img :src="pokemon.img" :alt="'Imagen de ' + pokemon.nombre" class="pokemon-image" />
          <div style="display: flex; justify-content: center; height: 10%; gap: 10%;">
            <div v-for="(tipo, i) in pokemon.tipos" :key="i" :style="'background-color: ' + colorestipo[tipo]" class="pokemon-type">
              <p>{{ tipo }}</p>
            </div>
          </div>
        </button>
      </div>
    </div>

    <div class="card" v-if="filtronombre == true">
        <button  class="card-content" @click="abrirDetalle(index)">
          <div class="card-number">#{{ pokemonBuscado.numero }}</div>
          <div class="card-name">{{ pokemonBuscado.nombre }}</div>
          <img :src="pokemonBuscado.img" :alt="'Imagen de ' + pokemonBuscado.nombre" class="pokemon-image" />
          <div style="display: flex; justify-content: center; height: 10%; gap: 10%;">
            <div v-for="(tipo, i) in pokemonBuscado.tipos" :key="i" :style="'background-color: ' + colorestipo[tipo]" class="pokemon-type">
              <p>{{ tipo }}</p>
            </div>
          </div>
        </button>
      </div>

    

  </div>

  <!-- Botón para cargar más Pokémon -->
  <div class="divbutonmas">  <button class="mostrarmas" @click="mostrarMas">Mostrar más</button></div>


  <!-- Modal -->
  <div class="modal" v-if="mostrarModal">
    <div class="modal-content">
      <div style="display: flex; justify-content: space-between;">
        <div>#{{ detallePokemon.numero }}</div>
        <button @click="cerrarDetalle" style="background-color: transparent;">❌</button>
      </div>

      <h2>{{ detallePokemon.nombre }}</h2>
      <div>Altura: {{ detallePokemon.estadistica }}M</div>
      <div>Peso: {{ detallePokemon.peso }}kg</div>
      <img :src="detallePokemon.img" :alt="'Imagen de ' + detallePokemon.nombre" class="modal-image" />
      <table>
            <tr>
              <td>HP</td>
              <td><progress :value="detallePokemon.hp" max="100"></progress>{{ detallePokemon.hp }}</td>
            </tr>
            <tr>
              <td>Ataque</td>
              <td><progress :value="detallePokemon.ataque" max="100"></progress>{{ detallePokemon.ataque }}</td>
            </tr>
            <tr>
              <td>Defensa</td>
              <td><progress :value="detallePokemon.defensa" max="100"></progress>{{ detallePokemon.defensa }}</td>
            </tr>
            <tr>
              <td>Ataque Especial</td>
              <td><progress :value="detallePokemon.as" max="100"></progress>{{ detallePokemon.as }}</td>
            </tr>
            <tr>
              <td>Defensa Especial</td>
              <td><progress :value="detallePokemon.sd" max="100"></progress>{{ detallePokemon.sd }}</td>
            </tr>
            <tr>
              <td>Velocidad</td>
              <td><progress :value="detallePokemon.s" max="100"></progress>{{ detallePokemon.s }}</td>
            </tr>
          </table>
    </div>
  </div>
</template>

<script setup>
import axios from "axios";
import { ref, onMounted } from "vue";

const pokemons = ref([]);
const mostrarModal = ref(false);
const detallePokemon = ref({});
const colorestipo = {
  grass: "#4CAF50",
  poison: "#800080",
  fire: "#FF0000",
  flying: "#00BFFF",
  water: "#2196F3",
  bug: "#6B8E23",
  normal: "#808080",
  electric: "#D7CD04",
  ground: "#8B4513",
  fairy: "#FF69B4",
  ghost: "#4B0082",
  steel: "#A9A9A9",
  dark: "#2F4F4F",
  dragon: "#483D8B",
  rock: "#A0522D",
  psychic: "#FF1493",
  ice: "#00CED1"
};
const numeroDeCartas = ref(20); // Inicialmente, muestra 20 cartas
const filtropokemones = ref([]);

async function obtenerurlpokemon() {
  for (let i = 1; i <= 1200; i++) {
    const response = await axios.get(`https://pokeapi.co/api/v2/pokemon/${i}`);
    const r = response.data;

    pokemons.value.push({
      numero: r.id,
      nombre: r.name.toUpperCase(),
      estadistica: r.height,
      peso: r.weight,
      hp: Math.min(Math.max(r.stats[0].base_stat, 0), 100),
      ataque: r.stats[1].base_stat,
      defensa: r.stats[2].base_stat,
      as: r.stats[2].base_stat,
      sd: r.stats[4].base_stat,
      s: r.stats[5].base_stat,
      img: r.sprites.other['official-artwork'].front_default,
      mostrarDetalle: false,
      tipos: r.types.map((e) => e.type.name)
    });
  }
}

function abrirDetalle(index) {
  console.log(index);
  console.log(filtropokemones.value);
  if(activarfiltro.value==true){
    detallePokemon.value = filtropokemones.value[index]
  }else detallePokemon.value = pokemons.value[index];
  mostrarModal.value = true;
}

function cerrarDetalle() {
  mostrarModal.value = false;
}

function mostrarMas() {
  numeroDeCartas.value += 50; // Incrementa el número de cartas a mostrar
}

onMounted(() => {
  obtenerurlpokemon();
});

const pokemonBuscado = ref({});
const nombrepokemon = ref("");
const filtronombre = ref(false);

function filtrarPokemonPorNombre() {
  console.log(nombrepokemon.value);
  pokemons.value.forEach((pokemon) => {
    if (pokemon.nombre === nombrepokemon.value.toUpperCase()) {
      pokemonBuscado.value = pokemon;
      filtronombre.value = true;
    }
  });
}


const activarfiltro = ref(false);
const tipoSeleccionado = ref("Todos")

function filtrarpokemon() {
  if (tipoSeleccionado.value == "Todos") {
    activarfiltro.value = false;
    return;
  }

  filtropokemones.value = [];
  pokemons.value.forEach((pokemon) => {
    if (pokemon.tipos.includes(tipoSeleccionado.value)) {
      filtropokemones.value.push(pokemon);
    }
  });

  activarfiltro.value = true;
}

</script>

<style scoped>
body{
  margin: 0px;
}
.principal {
  background: black;
}

.diseño{
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.arriba{
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 250px;
  background: red;
}

.foto{
  width: 50%;
}

.buscar{
  display: flex;
  align-items: center;
  justify-content: center;
  width: 50%;
}

.recto{
  display: flex;
  align-items: center;
  width: 100%;
  height: 100px;
  background: rgb(0, 0, 0);
}

.bola{
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 100%;
  width: 200px;
  height: 200px;
  background-color: rgb(0, 0, 0);
  position: absolute;
  top: 33%;
 
}

.bolapequeña{
  width: 150px ;
  height: 150px;
  background-color: white;
  border-radius: 100%;
}

.abajo{
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 250px;
  background: rgb(255, 255, 255);
}

.cartas{
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100%;
  background: linear-gradient(white, red);
  gap: 20px;
}

.card {
  margin-top: 10px;
  margin-bottom: 10px;
  width: 300px;
  background-color: #000000;
  border-radius: 10px;
  padding: 8px;
  text-align: center;
}

.card:hover{
  box-shadow: 0px 0px 10px 0px rgb(80, 126, 9);
}


.card-number {
  font-size: 30px;
  color: #007bff;
}

.card-name {
  font-size: 20px;
  margin-top: 5px;
  margin-bottom: 8px;
}

button {
  background-color: #ffffff38;
  color: white;
  border: none;
  padding: 5px 10px;
  cursor: pointer;
}

.pokemon-data {
  font-size: 14px;
}
.pokemon-image {
  max-width: 100%;
  display: block;
  margin: 0 auto;
  margin-top: 10px;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 999;
}

.modal-content {
  background-color: white;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.3);
  text-align: center;
}

.modal button {
  background-color: #007bff;
  color: white;
  border: none;
  padding: 5px 10px;
  cursor: pointer;
}

.modal-image {
  max-width: 100%;
  height: auto;
  margin-top: 10px;
}
.pokemon-type {
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 5px;
  width: 30%;
  height: 30px;
  font-size: 15px;
  text-align: center;
  color: rgb(0, 0, 0);
  font-weight: bold;
  text-transform: uppercase;
}

.divbutonmas{
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 50px;
}

.mostrarmas{
  height: 100%;
  width: 100%;
  background: #ffffff;
  color: rgb(0, 0, 0);
}

.mostrarmas:hover{
  background: rgb(0, 0, 0);
  font-size: 25px;
  color: rgb(255, 255, 255);
}
table{
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
td{
  padding: 10px;
}
</style>