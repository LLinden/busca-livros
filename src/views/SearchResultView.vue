<script setup>
import { useRoute } from 'vue-router'
import axios from 'axios'
import { computed, onMounted, ref } from 'vue'
import 'vue-loading-overlay/dist/css/index.css'
import Loading from 'vue-loading-overlay'

// useRoute retorna a rota atual ativa, muda automaticamente se a URL mudar
// é para ler parâmetros, query params, e o caminho atual
const route = useRoute()

const searchString = route.query.searchString
const books = ref([])
const currentPageCount = ref(0)
const totalBooks = ref(0)
// para ir e voltar para próxima página
const previousEndpoint = ref(null)
const nextEndpoint = ref(null)
const isLoading = ref(true)

// função para se comunicar com o serviço da API de livros
async function getBookData(url) {
    isLoading.value = true

    // faz a página ficar em branco antes de carregar os livros
    books.value = []

    const result = await axios.get(url);

    // salva os resultados da chamada acima na lista de livros
    books.value = result.data.results
    // salva quantas páginas de retorno
    currentPageCount.value = books.value.length
    // salva quantos livros retornaram
    totalBooks.value = result.data.count
    previousEndpoint.value = result.data.previous
    nextEndpoint.value = result.data.next

    // para saber quando acabou de carregar os livros
    isLoading.value = false
}

// estrai a página da url do endpoint
function extractPageFromUrl(url, sum) {
    const page = url.split("&page=").slice(-1)[0].split('&')[0]
    return parseInt(page) + sum
}

// define qual a página atual
const currentPage = computed(() => {
    if (nextEndpoint.value) {
        return extractPageFromUrl(nextEndpoint.value, -1)
    } else if (previousEndpoint.value) {
        return extractPageFromUrl(previousEndpoint.value, 1)
    }

    return 1
})

// para executar a função no momento correto
onMounted(async () => {
    await getBookData(`https://gutendex.com/books/?search=${searchString}&limit=10$copyright=false&mime_type=image/jpeg`)
})

</script>

<template>
    <div>
        <!-- loading da biblioteca vue-loading-overlay -->
        <loading :active="isLoading" is-full-page="true"></loading>
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
            <v-row align="center" justify="center">
                <v-btn @click="getBookData(previousEndpoint)" :disabled="previousEndpoint == null">
                    Anterior
                </v-btn>
                <v-btn @click="getBookData(nextEndpoint)" :disabled="nextEndpoint == null">
                    Próxima
                </v-btn>
            </v-row>
            <v-row :key="book.id" v-for="book in books">
                <v-col class="v-col-3">
                    <img :src="book.formats['image/jpeg']">
                </v-col>
                <v-col class="v-col-7">
                    <h2>{{ book.title }}</h2>
                    <h3> Autores </h3>
                    <ul>
                        <li class="authors-list" :key="author.name" v-for="author in book.authors">
                            {{ author.name }}
                        </li>
                    </ul>
                    <div class="download-area">
                        <p v-if="book.formats['application/epub+zip']">
                            <a :href="book.formats['application/epub+zip']">
                                Download
                            </a>
                        </p>
                        <p v-else>
                            Download Indisponível
                        </p>
                    </div>
                </v-col>
            </v-row>
        </v-container>
    </div>
</template>

<style scoped>
.authors-list {
    margin-left: 20px;
    list-style-type: none;
}

.download-area {
    margin-top: 30px;
}
</style>