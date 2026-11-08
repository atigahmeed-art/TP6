<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'

interface Pays {
  id: string
  nom: string
}
interface Ville {
  id: number
  nom: string
}

/* ===== PARTIE 1 : recherche pays par id ===== */
const idPays = ref('')
const nomPays = ref('')
const erreur = ref('')
const chargement = ref(false)

async function envoyer() {
  nomPays.value = ''
  erreur.value = ''

  if (!idPays.value.trim()) {
    erreur.value = 'Veuillez saisir un id de pays'
    return
  }

  chargement.value = true
  try {
    const reponse = await fetch(
      `http://localhost/api/pays.php?id=${idPays.value.trim()}`
    )
    if (!reponse.ok) throw new Error('Pays introuvable')

    const data = await reponse.json()
    nomPays.value = data.nom
  } catch (e: any) {
    erreur.value = e.message
  } finally {
    chargement.value = false
  }
}

/* ===== PARTIE 2 : select pays -> select villes ===== */
const listePays = ref<Pays[]>([])
const listeVilles = ref<Ville[]>([])
const paysChoisi = ref('')
const villeChoisie = ref('')
const erreur2 = ref('')

onMounted(async () => {
  try {
    const reponse = await fetch('http://localhost/api/liste_pays.php')
    listePays.value = await reponse.json()
  } catch (e) {
    erreur2.value = 'Impossible de charger les pays'
  }
})

watch(paysChoisi, async (id) => {
  listeVilles.value = []
  villeChoisie.value = ''

  if (!id) return

  try {
    const reponse = await fetch(`http://localhost/api/villes.php?pays_id=${id}`)
    listeVilles.value = await reponse.json()
  } catch (e) {
    erreur2.value = 'Impossible de charger les villes'
  }
})
</script>

<template>
  <main>
    <!-- PARTIE 1 -->
    <section>
      <h1>Rechercher un pays</h1>

      <input
        v-model="idPays"
        type="text"
        placeholder="Ex : TN, FR, DE"
        @keyup.enter="envoyer"
      />
      <button @click="envoyer" :disabled="chargement">Envoyer</button>

      <p v-if="chargement">Chargement...</p>
      <p v-if="nomPays">Nom du pays : <strong>{{ nomPays }}</strong></p>
      <p v-if="erreur" style="color: red">{{ erreur }}</p>
    </section>

    <hr />

    <!-- PARTIE 2 -->
    <section>
      <h1>Pays et villes</h1>

      <label>Pays :</label>
      <select v-model="paysChoisi">
        <option value="">-- Choisir un pays --</option>
        <option v-for="p in listePays" :key="p.id" :value="p.id">
          {{ p.nom }}
        </option>
      </select>

      <br /><br />

      <label>Ville :</label>
      <select v-model="villeChoisie" :disabled="!paysChoisi">
        <option value="">-- Choisir une ville --</option>
        <option v-for="v in listeVilles" :key="v.id" :value="v.id">
          {{ v.nom }}
        </option>
      </select>

      <p v-if="villeChoisie">
        Ville sélectionnée :
        <strong>{{ listeVilles.find(v => v.id == Number(villeChoisie))?.nom }}</strong>
      </p>
      <p v-if="erreur2" style="color: red">{{ erreur2 }}</p>
    </section>
  </main>
</template>