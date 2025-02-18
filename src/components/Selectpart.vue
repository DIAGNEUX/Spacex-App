<template>
  <div class="bg-black text-white py-5 flex items-center justify-center">
    <div class="container w-full max-w-3xl mx-auto rounded-lg">
      <!-- Titre au-dessus de wrap-select -->
      <h2 class="text-4xl font-bold text-center mb-5">Liste des lancements</h2>

      <div class="bg-black border border-gray-800 p-5 rounded-lg">
        <!-- Filtre -->
        <div class="flex justify-end items-center gap-4 mb-5">
          <label for="filtre" class="text-sm">Filtrer :</label>
          <select id="filtre" v-model="filtreActif" class="bg-black text-white p-2 rounded-md text-sm cursor-pointer border border-gray-800">
            <option value="tous">Tous les lancements</option>
            <option value="succes">Lancements réussis</option>
            <option value="echec">Lancements échoués</option>
          </select>
        </div>

        <!-- Tableau des lancements -->
        <table class="w-full table-auto border-collapse mb-5">
          <thead>
            <tr>
              <th class="text-left py-2 px-3 bg-gray-800 text-white">Nom</th>
              <th class="text-left py-2 px-3 bg-gray-800 text-white">Date</th>
              <th class="text-left py-2 px-3 bg-gray-800 text-white">Statut</th>
            </tr>
          </thead>
          <tbody>
            <tr class="border-b border-gray-800"  v-for="lancement in lancementsPagines" :key="lancement.id ">
              <td class="py-2 px-3 ">{{ lancement.nom }}</td>
              <td class="py-2 px-3">{{ formatDate(lancement.date) }}</td>
              <td :class="lancement.succes ? 'text-green-500 font-bold' : 'text-red-500 font-bold'" class="py-2 px-3">
                {{ lancement.succes ? 'Réussi' : 'Échoué' }}
              </td>

            </tr>
            
          </tbody>
        </table>

        <!-- Pagination alignée à droite -->
        <div class="flex justify-end items-center gap-4">
          <button @click="pagePrecedente" :disabled="pageActuelle === 1" class="py-2 px-4 bg-blue-600 text-white rounded-md cursor-pointer disabled:bg-gray-600">Précédent</button>
          <span class="text-sm">Page {{ pageActuelle }} / {{ totalPages }}</span>
          <button @click="pageSuivante" :disabled="pageActuelle === totalPages" class="py-2 px-4 bg-blue-600 text-white rounded-md cursor-pointer disabled:bg-gray-600">Suivant</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from "vue";

const lancements = ref([]);
const filtreActif = ref("tous");
const pageActuelle = ref(1);
const taillePage = 8;

// Récupération des lancements depuis l'API SpaceX
const fetchLancements = async () => {
  try {
    const response = await fetch("https://api.spacexdata.com/v5/launches");
    const data = await response.json();
    lancements.value = data.map(launch => ({
      id: launch.id,
      nom: launch.name,
      date: launch.date_utc,
      succes: launch.success
    }));
  } catch (error) {
    console.error("Erreur lors de la récupération des lancements :", error);
  }
};

onMounted(fetchLancements);

// Filtrer les lancements
const lancementsFiltres = computed(() => {
  if (filtreActif.value === "tous") return lancements.value;
  return lancements.value.filter(l => l.succes === (filtreActif.value === "succes"));
});

// Pagination
const totalPages = computed(() => Math.ceil(lancementsFiltres.value.length / taillePage));

const lancementsPagines = computed(() => {
  const debut = (pageActuelle.value - 1) * taillePage;
  return lancementsFiltres.value.slice(debut, debut + taillePage);
});

const pagePrecedente = () => {
  if (pageActuelle.value > 1) pageActuelle.value--;
};

const pageSuivante = () => {
  if (pageActuelle.value < totalPages.value) pageActuelle.value++;
};

// Formatage des dates
const formatDate = (dateStr) => {
  const date = new Date(dateStr);
  return date.toLocaleDateString("fr-FR", { day: "numeric", month: "long", year: "numeric" });
};
</script>
