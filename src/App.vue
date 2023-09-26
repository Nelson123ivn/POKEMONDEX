<template>
  <div class="principal">
    <div class="arriba">
      <img  class="fondo" src="./imagenes/fondo.jpg" alt="">
    </div>
    <div v-for="(pokemon, index) in pokemons" :key="index" class="card">
      <div class="card-content">
        <div class="card-number">#{{ pokemon.numero }}</div>
        <div class="card-name">{{ pokemon.nombre }}</div>
        <div>Altura: {{ pokemon.estadistica }}M</div>
        <div>Peso: {{ pokemon.peso }}kg</div>
        <img :src="pokemon.img" alt="Imagen de {{ pokemon.nombre }}" class="pokemon-image" />
        <div style="display: flex; justify-content: center; height: 10%; gap: 10%;">
          <div v-for="(tipo, i) in pokemon.tipos" :key="i" :style="'background-color: ' + colorestipo[tipo]" class="pokemon-type">
            <p>{{ tipo }}</p>
          </div>
        </div>
        <button @click="abrirDetalle(index)">VER MAS</button>
      </div>
    </div>
  </div>

  <!-- Modal -->
  <div class="modal" v-if="mostrarModal">
    <div class="modal-content">
      <button @click="cerrarDetalle">Cerrar</button>
      <h2>{{ detallePokemon.nombre }}</h2>
      <img :src="detallePokemon.img" :alt="'Imagen de ' + detallePokemon.nombre" class="modal-image" />
      <div>Altura: {{ detallePokemon.estadistica }}M</div>
      <div>Peso: {{ detallePokemon.peso }}kg</div>
      <!-- Agrega otras estadísticas y detalles aquí -->
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
  electric: "#FFFF00",
  ground: "#8B4513",
  fairy: "#FF69B4"
}

async function obtenerurlpokemon() {
  for (let i = 1; i <= 50; i++) {
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
  detallePokemon.value = pokemons.value[index];
  mostrarModal.value = true;
}

function cerrarDetalle() {
  mostrarModal.value = false;
}

onMounted(() => {
  obtenerurlpokemon();
});

</script>

<style scoped>
body{
  margin: 0px;
}
.principal {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  align-items: center;
  gap: 20px;
}

.arriba{
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 500px;
  background: red;
}

.fondo{
  width: 90%;
  height: 450px;
}

.card {
  width: 300px;
  background-color: #f0f0f0;
  border-radius: 10px;
  padding: 8px;
  text-align: center;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.3);
}

.card-content {
  margin-bottom: 8px;
}

.card-number {
  font-size: 20px;
  color: #007bff;
}

.card-name {
  font-size: 16px;
  margin-top: 5px;
  margin-bottom: 8px;
}

button {
  background-color: #007bff;
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
  border-radius: 5px;
  width: 30%;
  height: 30px;
  font-size: 12px;
  text-align: center;
  color: white;
  font-weight: bold;
}
</style>