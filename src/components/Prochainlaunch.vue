<template>
  <div class="wrap-prochain-lancement">
    <div class="flex flex-col items-center justify-center w-full h-screen">
      <h2 class="text-5xl font-bold text-gray-300 mb-4">Prochain Lancement</h2>

      <div class=" border border-gray-300 shadow-lg rounded-lg p-6 text-center">
        <h3 class="text-2xl font-semibold text-gray-300">{{ nomLancement }}</h3>
        <p class="text-lg text-gray-300 my-3">{{ formatDate(dateLancement) }}</p>
        <div class="flex space-x-4 p-4 bg-white/10 rounded-lg">
          <!-- Jours -->
          <div class="flex flex-col items-center text-gray-300">
            <span class="text-8xl font-bold leading-none">{{ jours }}</span>
            <small class="text-lg text-white/70">Jours</small>
          </div>
          <!-- Deux-points alignés -->
          <div class="flex flex-col ">
            <span class="text-8xl font-bold text-gray-300 leading-none">:</span>
          </div>

          <!-- Heures -->
          <div class="flex flex-col items-center text-gray-300">
            <span class="text-8xl font-bold leading-none">{{ heures }}</span>
            <small class="text-lg text-white/70">Heures</small>
          </div>
          <!-- Deux-points alignés -->
          <div class="flex flex-col">
            <span class="text-8xl font-bold text-gray-300 leading-none">:</span>
          </div>

          <!-- Minutes -->
          <div class="flex flex-col items-center text-gray-300">
            <span class="text-8xl font-bold leading-none">{{ minutes }}</span>
            <small class="text-lg text-white/70">Minutes</small>
          </div>
          <!-- Deux-points alignés -->
          <div class="flex flex-col ">
            <span class="text-8xl font-bold text-gray-300 leading-none">:</span>
          </div>

          <!-- Secondes -->
          <div class="flex flex-col items-center text-gray-300">
            <span class="text-8xl font-bold leading-none">{{ secondes }}</span>
            <small class="text-lg text-white/70">Secondes</small>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from "vue";

// Données du lancement
const nomLancement = ref("Ariane 62 | VA263");
const dateLancement = ref(new Date("2025-02-26T16:24:00Z"));

// Décompte en temps réel
const jours = ref(0);
const heures = ref(0);
const minutes = ref(0);
const secondes = ref(0);
let interval: number;

const calculerDecompte = () => {
  const maintenant = new Date().getTime();
  const lancement = dateLancement.value.getTime();
  const diff = lancement - maintenant;

  if (diff <= 0) {
    jours.value = 0;
    heures.value = 0;
    minutes.value = 0;
    secondes.value = 0;
    clearInterval(interval);
    return;
  }

  jours.value = Math.floor(diff / (1000 * 60 * 60 * 24));
  heures.value = Math.floor((diff % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));
  minutes.value = Math.floor((diff % (1000 * 60 * 60)) / (1000 * 60));
  secondes.value = Math.floor((diff % (1000 * 60)) / 1000);
};

onMounted(() => {
  calculerDecompte();
  interval = setInterval(calculerDecompte, 1000);
});

onUnmounted(() => {
  clearInterval(interval);
});

const formatDate = (date: Date) => {
  return date.toLocaleString("fr-FR", {
    weekday: "long",
    day: "numeric",
    month: "long",
    year: "numeric",
    hour: "2-digit",
    minute: "2-digit",
    second: "2-digit",
    timeZoneName: "short",
  });
};
</script>

<style scoped>
.wrap-prochain-lancement {
  background-image: url('@/assets/bg-spacex.png');
  background-size: cover;
  background-position: center;
  height: 100vh;
}
</style>
