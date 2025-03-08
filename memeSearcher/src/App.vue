<script setup>
import { ref, computed } from "@vue/reactivity";

// Estado de memes y favoritos
const memes = ref([]);
let memesTotal = [];
const favorites = ref([]); // Inicializamos como un array vacío

// Cargar los memes desde la API
const loadData = async () => {
  const response = await fetch("https://api.imgflip.com/get_memes");
  const { data } = await response.json();

  memes.value = data.memes;
  memesTotal = data.memes;
  
  // Recuperamos los favoritos del almacenamiento local (si existen)
  const storedFavorites = JSON.parse(localStorage.getItem('favorites')) || [];
  favorites.value = storedFavorites;
};

// Guardar los favoritos en el almacenamiento local
const saveFavorites = () => {
  localStorage.setItem('favorites', JSON.stringify(favorites.value));
};

// Función para agregar o quitar de favoritos
const toggleFavorite = (meme) => {
  const index = favorites.value.findIndex(fav => fav.id === meme.id);
  
  if (index !== -1) {
    // Si ya está en favoritos, lo eliminamos
    favorites.value.splice(index, 1);
  } else {
    // Si no está, lo agregamos
    favorites.value.push(meme);
  }
  
  // Guardamos los favoritos después de modificar
  saveFavorites();
};

// Computed para verificar si un meme está en favoritos
const isFavorite = (memeId) => {
  return favorites.value.some(fav => fav.id === memeId);
};

// Ordenamos los memes: los favoritos primero
const sortedMemes = computed(() => {
  return [...memes.value].sort((a, b) => {
    const isFavoriteA = favorites.value.some(fav => fav.id === a.id);
    const isFavoriteB = favorites.value.some(fav => fav.id === b.id);
    
    // Los favoritos deben ir primero
    if (isFavoriteA && !isFavoriteB) return -1;
    if (!isFavoriteA && isFavoriteB) return 1;
    return 0; // Si ambos son favoritos o ninguno lo es, mantiene el orden original
  });
});

loadData();
</script>

<template>
  <div class="hero">
    <div class="hero-content text-center flex flex-col items-center">
      <h1 class="text-5xl font-bold mb-4">Meme Search</h1>
      <label class="input">
        <svg
          class="h-[1em] opacity-50"
          xmlns="http://www.w3.org/2000/svg"
          viewBox="0 0 24 24"
        >
          <g
            stroke-linejoin="round"
            stroke-linecap="round"
            stroke-width="2.5"
            fill="none"
            stroke="currentColor"
          >
            <circle cx="11" cy="11" r="8"></circle>
            <path d="m21 21-4.3-4.3"></path>
          </g>
        </svg>
        <input type="search" class="grow" placeholder="Search Meme" v-on:input="searchMeme" />
      </label>
    </div>
  </div>
  <section class="container mx-auto px-10 grid grid-cols-2 md:grid-cols-3 lg:grid-cols-5 gap-x-5 gap-y-4">
    <div 
      v-for="meme in sortedMemes" 
      :key="meme.id" 
      class="bg-white rounded-lg shadow-lg overflow-hidden relative"
    >
      <figure>
        <img :src="meme.url" alt="Meme" class="w-full h-auto" />
        
        <!-- Corazón para marcar como favorito -->
        <button 
          @click="toggleFavorite(meme)" 
          class="absolute top-2 right-2 z-10"
        >
          <!-- Usamos clases de daisyUI para el corazón con transición -->
          <svg
            xmlns="http://www.w3.org/2000/svg"
            class="w-8 h-8 inline-block transition-all duration-300 ease-in-out"
            :class="{
              'text-gray-500 stroke-black': !isFavorite(meme.id),  // Borde negro y fondo transparente
              'text-white fill-red-500 stroke-black': isFavorite(meme.id) // Fondo rojo y borde negro cuando está en favoritos
            }"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z"
            />
          </svg>
        </button>
      </figure>
      <div class="p-2 text-center">
        <h2 class="text-black font-bold">{{ meme.name }}</h2>
      </div>
    </div>
  </section>
</template>
