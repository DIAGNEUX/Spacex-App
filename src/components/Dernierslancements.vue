<template>
  <div class="bg-black text-white py-5">
    <div class="container mx-auto max-w-5xl">
      <!-- Liste des lancements -->
      <h2 class="text-4xl font-bold text-center mb-5">Derniers lancements</h2>

      <div class="flex flex-wrap gap-5 justify-center items-center p-2">
        <div
          v-for="lancement in lancementsFiltres.slice(0, 10)"
          :key="lancement.id"
          class="bg-gray-800 p-4 rounded-lg w-52 cursor-pointer hover:bg-gray-600 transition duration-300"
          @click="showModal(lancement)"
        >
          <p class="text-xs text-gray-400">{{ formatDate(lancement.date) }}</p>
          <h3 class="text-lg mt-2">{{ lancement.nom }}</h3>
        </div>
      </div>

      <!-- Modal -->
      <div v-if="modalVisible" class="fixed top-0 left-0 w-full h-full bg-black bg-opacity-70 flex justify-center items-center z-50" @click="modalVisible = false">
        <div class="bg-gray-800 text-white p-8 rounded-lg w-full max-w-4xl max-h-[90%] overflow-auto" @click.stop>
          <span class="absolute top-2 right-2 text-3xl cursor-pointer" @click="modalVisible = false">&times;</span>
          
          <div class="flex gap-8">
            <div class="flex-1">
              <h3 class="text-2xl font-bold">{{ selectedLancement.nom }}</h3>
              <p class="mt-2">{{ formatDate(selectedLancement.date) }}</p>
              <p class="mt-2">{{ selectedLancement.details }}</p>

              <!-- Détails  -->
              <p class="mt-2"><strong>Lieu de lancement :</strong> {{ selectedLancement.lieu }}</p>
              <p class="mt-2"><strong>Payloads :</strong> {{ selectedLancement.payloads.join(', ') }}</p>
              <p class="mt-2"><strong>Clients :</strong> {{ selectedLancement.clients.join(', ') }}</p>
              <div class="mt-4">
                <a :href="selectedLancement.article" target="_blank" class="text-blue-400 underline">Lire l'article</a>
              </div>
            </div>
            
            <div class="flex-1">
              <img 
                :src="selectedLancement.image || selectedLancement.patch || 'spacex.webp'" 
                alt="Image de la mission" 
                class="w-full max-h-96 object-cover mt-4"
                @error="imageError"
              />
            </div>
          </div>

          <!-- Switch pour afficher la vidéo -->

          <div class="mt-6 flex items-center gap-4">
            <input type="checkbox" id="videoSwitch" v-model="showVideo" class="w-6 h-6 cursor-pointer"/>
            <label for="videoSwitch" class="font-bold">Afficher la vidéo</label>
          </div>

          <!-- Afficher la vidéo YouTube si activée -->
          <div v-if="showVideo" class="mt-4">
            <iframe
              :src="'https://www.youtube.com/embed/' + selectedLancement.youtube_id"
              frameborder="0"
              allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture"
              allowfullscreen
              class="w-full h-80"
            ></iframe>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from "vue";

const lancements = ref([]);
const modalVisible = ref(false);
const selectedLancement = ref({});
const showVideo = ref(false);
const filtreActif = ref("tous");
const pageActuelle = ref(1);
const taillePage = 10;

// Récupération des lancements depuis l'API SpaceX
const fetchLancements = async () => {
  try {
    const response = await fetch("https://api.spacexdata.com/v5/launches");
    const data = await response.json();
    lancements.value = data.map(launch => ({
      id: launch.id,
      nom: launch.name,
      date: launch.date_utc,
      succes: launch.success,
      details: launch.details || 'Aucun détail disponible',
      image: launch.links.flickr.original.length > 0 ? launch.links.flickr.original[0] : '',
      patch: launch.links.patch.large || launch.links.patch.small,
      article: launch.links.article || '#',
      youtube_id: launch.links.youtube_id || '',
      lieu: launch.launchpad ? launch.launchpad.name : 'Inconnu',
      payloads: launch.payloads ? launch.payloads.map(p => p.name) : ['Inconnu'],
      clients: launch.payloads ? launch.payloads.map(p => p.clients).flat() : ['Inconnu']
    }));
  } catch (error) {
    console.error("Erreur lors de la récupération des lancements :", error);
  }
};

onMounted(fetchLancements);

// Filtrage des lancements
const lancementsFiltres = computed(() => {
  if (filtreActif.value === "tous") return lancements.value;
  return lancements.value.filter(l => l.succes === (filtreActif.value === "succes"));
});

// Formatage des dates
const formatDate = (dateStr) => {
  const date = new Date(dateStr);
  return date.toLocaleDateString("fr-FR", { day: "numeric", month: "numeric", year: "numeric" });
};

// Affichage du modal
const showModal = (lancement) => {
  selectedLancement.value = lancement;
  modalVisible.value = true;
};
const imageError = () => {
  selectedLancement.value.image = 'spacex.webp'; 
};
</script>
