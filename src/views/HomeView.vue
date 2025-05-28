<script setup>
import { ref, watch } from "vue"
import { useRouter } from "vue-router"

// useRouter usado para navegar, ou seja, mudar de rota programaticamente
// oferece métodos como .push(), .replace(), .back(), etc.
const router = useRouter()
const searchString = ref("")
// canSearch inicia como false até algo ser digitado na busca
const canSearch = ref(false)

function search() {
  // criação da url de busca
  router.push({
    path: 'search',
    query: {
      searchString: searchString.value
    }
  })
}

// observa se o valor da string de busca é maior que dois
// se for canSearch passa para true e habilita botão de busca
watch(searchString, () => {
 canSearch.value = searchString.value.length > 2
})

</script>

<template>
  <v-app>
    <v-container>
      <v-row class="default-margin" align="center" justify="center">
        <h1>Pesquisa de Livros Gratuitos</h1>
      </v-row>
      <v-row class="default-margin" align="center" justify="center">
        <v-text-field class="w-75" label="Busque por algo..." v-model="searchString"></v-text-field>
      </v-row>
      <v-row class="default-margin" align="center" justify="center">
        <v-btn @click="search" :disabled="!canSearch">
          Buscar
        </v-btn>
      </v-row>
    </v-container>
  </v-app>
</template>

<style scoped></style>