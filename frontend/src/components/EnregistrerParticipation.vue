<template>
  <div class="card">
    <h2>Enregistrer une participation</h2>

    <form class="participation-form" @submit.prevent="enregistrerParticipation">
      <label for="personne" class="label">Personne</label>
      <select
        id="personne"
        v-model="selectedPersonne"
        required
        class="select"
      >
        <option value="" disabled>-- personne --</option>
        <option
          v-for="personne in personnes"
          :key="personne.matricule"
          :value="personne.matricule"
        >
          {{ personne.prenom }} {{ personne.nom }}
        </option>
      </select>

      <label for="projet" class="label">Project</label>
      <select
        id="projet"
        v-model="selectedProjet"
        required
        class="select"
      >
        <option value="" disabled>-- projet --</option>
        <option
          v-for="projet in projets"
          :key="projet.id"
          :value="projet.id"
        >
          {{ projet.nom }}
        </option>
      </select>

      <label for="role" class="label">Role</label>
      <input
        id="role"
        type="text"
        v-model="role"
        placeholder="Ex: developer"
        required
        class="input"
      />

      <label for="pourcentage" class="label">Percentage</label>
      <div class="slider-container">
        <input
          id="pourcentage"
          type="range"
          v-model.number="pourcentage"
          step="0.1"
          min="0"
          max="1"
          class="slider"
        />
        <span class="slider-value">{{ (pourcentage * 100).toFixed(0) }}%</span>
      </div>

      <button type="submit" class="btn-submit">Enregistrer</button>
    </form>

    <div v-if="errorMsg" class="error">{{ errorMsg }}</div>
    <div v-if="successMsg" class="success">{{ successMsg }}</div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

const personnes = ref([])
const projets = ref([])
const selectedPersonne = ref('')
const selectedProjet = ref('')
const role = ref('')
const pourcentage = ref(0.1)
const errorMsg = ref('')
const successMsg = ref('')

const chargerPersonnes = async () => {
  try {
    const resp = await axios.get('/api/personnes')
    personnes.value = resp.data._embedded
      ? resp.data._embedded.personnes
      : resp.data
  } catch (err) {
    errorMsg.value = 'Erreur chargement personnes : ' + err
  }
}

const chargerProjets = async () => {
  try {
    const resp = await axios.get('/api/projets')
    projets.value = resp.data._embedded
      ? resp.data._embedded.projets
      : resp.data
  } catch (err) {
    errorMsg.value = 'Erreur chargement projets : ' + err
  }
}

const enregistrerParticipation = async () => {
  const payload = {
    matricule: selectedPersonne.value,
    codeProjet: selectedProjet.value,
    role: role.value,
    pourcentage: pourcentage.value,
  }
  try {
    await axios.post('/api/gestion/participation', payload)
    successMsg.value = 'Participation enregistrée avec succès.'
    errorMsg.value = ''
    selectedPersonne.value = ''
    selectedProjet.value = ''
    role.value = ''
    pourcentage.value = 0.1
  } catch (err) {
    successMsg.value = ''
    if (err.response && err.response.data && err.response.data.message) {
      errorMsg.value = err.response.data.message
    } else {
      errorMsg.value = 'Erreur lors de l’enregistrement.'
    }
  }
}

onMounted(() => {
  chargerPersonnes()
  chargerProjets()
})
</script>

<style scoped>
.card {
  max-width: 400px;
  margin: 2rem auto;
  padding: 2rem;
  background-color: #fff;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
}

h2 {
  margin-bottom: 1.5rem;
  text-align: center;
  font-size: 1.25rem;
}
.participation-form {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
  background-color: #fff;
  padding: 2rem;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  width: 100%;
  max-width: 500px;
  margin: 0 auto;
}

.label {
  font-weight: 700;
  margin-bottom: 0.25rem;
}

.select {
  padding: 0.75rem 1rem;
  border: 1px solid #ccc;
  border-radius: 8px;
  background-color: #f9f9f9;
  font-size: 1rem;
  color: #333;
  appearance: none;
  cursor: pointer;
  transition: border-color 0.2s ease-in-out, background-color 0.2s ease-in-out;
  width: 100%;
}

.select:hover {
  border-color: #33ffb5;
  background-color: #f0fdf4;
}

.select:focus {
  border-color: #33ffb5;
  outline: none;
  background-color: #e8f7f0;
}

.select::-ms-expand {
  display: none;
}

.select::placeholder {
  color: #6b6b6b;
  font-style: normal;
  font-size: 0.9rem;
}

.select option {
  padding: 1rem;
  background-color: #fff;
  color: #333;
  border: none;
}

.select option:hover {
  background-color: #33ffb5;
  color: #fff;
}

.slider-container {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.slider {
  flex: 1;
  -webkit-appearance: none;
  height: 4px;
  background: #6b6b6b;
  border-radius: 2px;
  outline: none;
  cursor: pointer;
}

.slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 14px;
  height: 14px;
  border-radius: 50%;
  background: #33ffb5;
  cursor: pointer;
  border: 2px solid #fff;
  box-shadow: 0 0 2px rgba(0,0,0,0.2);
}

.slider::-moz-range-thumb {
  width: 14px;
  height: 14px;
  border-radius: 50%;
  background: #33ffb5;
  cursor: pointer;
  border: 2px solid #fff;
  box-shadow: 0 0 2px rgba(0,0,0,0.2);
}

.slider-value {
  width: 40px;
  text-align: center;
}

.btn-submit {
  padding: 0.4rem 0.8rem;
  background-color: #33ffb5;
  color: #fff;
  font-weight: 600;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  transition: background-color 0.2s ease-in-out;
  font-size: 0.8rem;
  margin-left: 0;
}

.btn-submit:hover {
  background-color: #33ffb5;
}

.participation-form {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
  align-items: flex-start;
}

::placeholder {
  color: #ffffff;
  font-size: 0.85rem;
  font-style: italic;
  opacity: 1;
}

.input::placeholder, .select::placeholder {
  color: #6b6b6b;
  font-style: normal;
  font-size: 0.9rem;
}

.error {
  margin-top: 1rem;
  color: #ff3f33;
}

.success {
  margin-top: 1rem;
  color: #61ff33;
}
</style>
