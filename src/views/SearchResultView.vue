<script setup>
import { useRoute } from 'vue-router'
import axios from 'axios'
import { computed, onMounted, ref } from 'vue'

// useRoute retorna a rota atual ativa, muda automaticamente se a URL mudar
// é para ler parâmetros, query params, e o caminho atual
const route = useRoute()

const searchString = route.query.searchString
const books = ref({})
const currentPageCount = ref(0)
const totalBooks = ref(0)
// para ir e voltar para próxima página
const previousEndpoint = ref(null)
const nextEndpoint = ref(null)

// Função para se comunicar com o serviço da API de livros
async function getBookData(searchValue) {
    const result = await axios.get(`https://gutendex.com/books/?search=${searchValue}&limit=10$copyright=false&mime_type=image/jpeg`);

    // salva os resultados da chamada acima na lista de livros
    books.value = result.data.results
    // salva quantas páginas de retorno
    currentPageCount.value = books.value.legth
    // salva quantos livros retornaram
    totalBooks.value = result.data.count
    previousEndpoint.value = result.data.previous
    nextEndpoint.value = result.data.next
}

// estrai a página da url do endpoint
function extractPageFromUrl(url, sum) {
    const page = url.split("&page=").slice(-1)[0].split('&')[0]
    return parseInt(page) + sum
}

// define qual a página atual
const currentPage = computed(() => {
    if (nextEndpoint.valeu) {
        return extractPageFromUrl(nextEndpoint.value, -1)
    } else if (previousEndpoint.value) {
        return extractPageFromUrl(previousEndpoint.value, 1)
    }

    return 1
})

// para executar a função no momento correto
onMounted(async () => {
    await getBookData(searchString)
})

</script>

<template>
    <v-container>
        <v-row align="center" justify="center">
            <h1>Vendo os resultados para busca "{{ searchString }}"</h1>
        </v-row>
        <v-row align="center" justify="center">
            <h2>Total de resultados: {{ totalBooks }}</h2>
        </v-row>
        <v-row align="center" justify="center">
            <h3>Página atual: {{ currentPage }}</h3>
        </v-row>
    </v-container>
</template>