<template>
  <div class="page">
    <div class="search">
      <div class="search-fields">
        <input type="text" class="search-field" placeholder="Type Movie name" v-model="movieSearchTerm" @keyup.enter="initiateSearch" />
        <input type="text" class="search-field" placeholder="Type TV Show name" v-model="tvSearchTerm" @keyup.enter="initiateSearch" />
      </div>
      <div class="search-btn">
        <button @click="initiateSearch">Search</button>
      </div>
      <div class="page-btn" v-if="movies.length || tv.length">
        <button class="prev" v-if="currentPage > 2" @click="currentPage--; changePage()">Previous Page</button>
        <button class="prev" v-if="currentPage === 2" @click="currentPage--; changePage()">First Page</button>
        <button class="next" v-if="currentPage === totalPages-1" @click="currentPage++; changePage()">Last Page</button>
        <button class="next" v-if="currentPage < totalPages-1" @click="currentPage++; changePage()">Next Page</button>
      </div>
      <div class="results" v-if="movies.length">
        <ul class="results-list">
          <li v-for="(result, index) in movies" :key="index">
            <button class="movie-link" @click="idMovie = result.id">
              <img class="movie-link-img" v-if="result.poster_path" :src="`https://image.tmdb.org/t/p/w154${result.poster_path}`"/>
              <div class="no-poster" v-else>
                <img class="no-poster-img-main" src="./assets/no-poster.png" />
                <p class="no-poster-title">{{ result.title }}</p>
              </div>
            </button>
          </li>
        </ul>
      </div>
      <div class="results" v-if="tv.length">
        <ul class="results-list">
          <li v-for="(result, index) in tv" :key="index">
            <button class="movie-link" @click="idTv = result.id">
              <img class="movie-link-img" v-if="result.poster_path" :src="`https://image.tmdb.org/t/p/w154${result.poster_path}`"/>
              <div class="no-poster" v-else>
                <img class="no-poster-img-main" src="./assets/no-poster.png" />
                <p class="no-poster-title">{{ result.name }}</p>
              </div>
            </button>
          </li>
        </ul>
      </div>
      <p v-if="noResults && !movies.length && !tv.length">No results!</p>
    </div>
    <Movie class="card" @close="close" v-if="idMovie" :idMovie="idMovie" :apiKey="apiKey" />
    <Tv class="card" @close="close" v-if="idTv" :idTv="idTv" :apiKey="apiKey" />
    <ApiKey class="card" @apiInput="key => apiKey = key" v-if="!apiKey" />
  
  </div>
</template>

<script>
import { defineComponent, ref } from 'vue';
import Movie from './components/Movie.vue';
import Tv from './components/Tv.vue';
import ApiKey from './components/ApiKey.vue'

export default defineComponent({
  components: {
    Movie,
    Tv,
    ApiKey,
  },
  setup: () => {
    const tvSearchTerm = ref('');
    const movieSearchTerm = ref('');
    const cachedSearchTerm = ref('');
    const movies = ref([]);
    const tv = ref([]);
    const noResults = ref(false);
    const currentPage = ref(1);
    const totalPages = ref();
    const idMovie = ref();
    const idTv = ref();
    const apiKey = ref(localStorage.getItem('apiKey') ?? '');

    const initiateSearch = async () => {
      if (movieSearchTerm.value) {
        tv.value.length = 0;
        currentPage.value = 1;
        const response = await fetch(`https://api.themoviedb.org/3/search/movie?api_key=${apiKey.value}&query=${movieSearchTerm.value}&page=${currentPage.value}`)
          .then(async response => response.json())
        movies.value = response.results;
        cachedSearchTerm.value = movieSearchTerm.value
        pageSetup(response)
      } else if (tvSearchTerm.value) {
        movies.value.length = 0;
        currentPage.value = 1;
        const response = await fetch(`https://api.themoviedb.org/3/search/tv?api_key=${apiKey.value}&query=${tvSearchTerm.value}&page=${currentPage.value}`)
          .then(async response => response.json())
        tv.value = response.results;
        cachedSearchTerm.value = tvSearchTerm.value

        pageSetup(response)
      } else return
    };

    const pageSetup = (response) => {
      totalPages.value = response.total_pages;
      movieSearchTerm.value = '';
      tvSearchTerm.value = '';
      noResults.value = response.total_results === 0;
    }

    const changePage = async () => {
      if (movies.value.length) {
        const response = await fetch(`https://api.themoviedb.org/3/search/movie?api_key=${apiKey.value}&query=${cachedSearchTerm.value}&page=${currentPage.value}`)
          .then(async response => response.json())
        movies.value = response.results;
        return
      }
      const response = await fetch(`https://api.themoviedb.org/3/search/tv?api_key=${apiKey.value}&query=${cachedSearchTerm.value}&page=${currentPage.value}`)
        .then(async response => response.json())
      tv.value = response.results;
    };

    const close = () => {
      idMovie.value = undefined
      idTv.value = undefined
    }

    return {
      movieSearchTerm,
      tvSearchTerm,
      movies,
      tv,
      idMovie,
      idTv,
      apiKey,
      initiateSearch,
      changePage,
      close,
      noResults,
      currentPage,
      totalPages,
    }
  }
});

</script>

<style>
@font-face {
  font-family: Stencil;
  src: url("./assets/StencilStd/StencilStd.otf") format("opentype");
}

html, .movie-info, .input-card {
  background-color: #370101
}

.search-field, .search > p, span, .search-btn, .page-btn, .countries, .name, .character, .api-input-field, .api-sender {
  font-family: 'Raleway', sans-serif;
  color: #a88100;
}

.input-card, .search, .search-fields, .search-btn, .page-btn, .results-list, .card, .movie-cols, .poster, .cast-members, .links, .name-div, .character {
  display: flex;
}

.input-card, .search {
    flex-direction: column;
}

.page, .no-poster, .movie-cols, .poster, .cast-member {
  position: relative;
}

.no-poster-title, .card-no-poster-title, .character {
  position: absolute;
  text-align: center;
}

.search, .movie-title, .cast {
  margin: 0 auto;
}

.search-field, .api-input-field, .countries, .page-btn > *,
.search-btn > *, .api-sender, .name-div {
  background-color: #752f00;
  border-radius: 5px;
}

.input-card, .search-field, .search-btn > button, .page-btn > button, .api-sender {
  padding: 5px
}

.search {
  width: 760px;
  padding-top: 5px;
}

.api-sender {
    align-self: center;
    margin-top: 5px;
}

.search-fields, .card, .api-card {
  width: 100%;
}

.input-card, .api-sender, .search-field, .search-btn > button {
  width: 40%;
}

.search-field:focus, .api-input-field:focus, .countries:focus {
  outline: 2px solid #a88100;
}

.search-field, .search-btn > button, .page-btn > button, .api-sender {
  margin-bottom: 5px;
}

.movie-link-img, .movie-link, .no-poster, .no-poster-title, .countries {
  width: 150px;
}

.search-fields, .page-btn {
  justify-content: space-between;
}

.page-btn > *,
.search-btn > *, .movie-link, .api-sender, .name-div, .cast-member {
  transition: 250ms linear;
}

.page-btn > *:hover, .search-btn > *:hover, .api-sender:hover, .movie-link:hover, .cast-member:hover {
  transform: scale(1.05);
}

.input-card, .search-btn, .card, .movie-cols, .api-card {
  justify-content: center;
}

.results {
  padding-top: 15px;
}

.results-list, .cast-members {
  flex-wrap: wrap;
}

.movie-link-img, .movie-link {
  height: 225px;
}

.movie-link {
  margin: -6px 2px 8px 0;
}

.no-poster {
  z-index: -1;
}

.no-poster-title {
  font-family: "Stencil", Times, serif;
  top: 10px;
  font-size: 14px;
}

.previous, .imdb {
  margin-right: auto;
}

.next, .website {
  margin-left: auto;
}

.card, .api-card {
  position: fixed;
  top: 0;
  right: 0;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
}

.input-card, .movie-cols {
  overflow: hidden;
  height: fit-content;
  margin: 10%;
  border-radius: 10px;
  box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);
}

.movie-info {
  padding: 10px;
  width: 365px;
  height: 750px;
  overflow: auto;
}

.movie-info > * {
  padding-bottom: 6px;
  font-family: 'Lato', sans-serif;
  color: #a88100;
}

.links {
  text-decoration: underline;
}

.movie-poster,
.no-poster-img {
  border-top-left-radius: 10px;
  border-bottom-left-radius: 10px;
}

.card-no-poster-title {
  font-family: "Stencil", Times, serif;
  top: 50px;
  width: 500px;
  padding: 10px;
  font-size: 40px;
}

.cast {
  padding-top: 10px;
}

.cast-member, .name-div, .name, .character, .character > p {
  align-items: center;
  width: 80px;
}

.cast-member {
  margin: -6px 2px 8px 0;
  cursor: auto;
}

.cast-member:hover {
  z-index: +1;
}

.name-div {
  height: 50px;
}

.name, .character, .character > p {
  font-size: 12px;
  word-wrap: break-word;
}

.character {
  color:#ffb514;
  opacity: 0;
  top: 0;
  height: 170px;
  background-color: rgba(117,47,0, 0.5);
}

.character:hover {
  opacity: 1;
  transition: opacity 500ms;
}
</style>