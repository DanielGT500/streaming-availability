<template>
    <div class="card" @click="$emit('close')">
        <div class="movie-cols" @click.stop="" v-if="movie">
            <div class="poster">
                <img v-if="movie.poster_path" class="movie-poster" :src="`https://image.tmdb.org/t/p/w500${movie.poster_path}`" />
                <div v-else>
                    <img class="no-poster-img" src="../assets/no-poster.png" />
                    <p class="card-no-poster-title">{{ movie.title }}</p>
                </div>
            </div>
            <div class="movie-info">
                <div class="links">
                    <a v-if="movie.imdb_id" class="imdb" :href="`https://www.imdb.com/title/${movie.imdb_id}`" target="blank" ><span>IMDB Link</span></a>
                    <a v-if="movie.homepage" class="website" :href="`${movie.homepage}`" target="blank"><span>Official Website</span></a>
                </div>
                <p class="provider" v-if="countries.length">
                    <select v-model="countryCode" name="countries" class="countries" required>
                        <option value="placeholder" disabled selected hidden>Streaming on</option>
                        <option v-for="(country, index) in countries" :value="country.code" :key="index">{{ country.name }}</option>
                    </select>
                    <br> {{ movie["watch/providers"].results[countryCode]?.flatrate.map(x => x.provider_name).join(', ') }}
                </p>
                <p v-if="movie.genres.length"><span>Genre(s)</span><br> {{ movie.genres.map(x => x.name).join('/') }}</p>
                <p v-if="movie.release_date"><span>Release date</span><br> {{ movie.release_date }}</p>
                <p v-if="movie.runtime && (movie.runtime > 0)"><span>Runtime</span><br> {{ movie.runtime }} minutes</p>
                <p v-if="movie.budget && (movie.budget > 0)"><span>Budget</span><br> {{ '$' + movie.budget.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ',') }}</p>
                <p v-if="movie.revenue && (movie.revenue > 0)"><span>Revenue</span><br> {{ '$' + movie.revenue.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ',') }}</p>
                <p v-if="movie.overview"><span>Overview</span><br> {{ movie.overview}} </p>
                <p v-if="movie.credits.cast.length"><span>Cast</span></p>
                <div v-if="movie.credits.cast.length" class="cast">
                    <ul class="cast-members">
                        <li v-for="(castMember, index) in cast" :key="index">
                            <button class="cast-member">
                                <img :src="`https://image.tmdb.org/t/p/w185${castMember.profile_path}`" v-if="castMember.profile_path">
                                <img src="../assets/no-profile.png" v-else>
                                <div class="name-div"><p class="name">{{ castMember.name }}</p></div>
                                <div class="character" v-if="castMember.character"><p>As:<br>{{castMember.character}}</p></div>
                            </button>
                        </li>
                    </ul>
                </div>
                <p v-if="!movie.imdb_id && !movie.homepage && !countries.length && !movie.genres.length && 
                !movie.release_date && !movie.runtime && !movie.budget && !movie.revenue && 
                !movie.overview && !movie.credits.cast.length && !(movie.runtime > 0) && 
                !(movie.budget > 0) && !(movie.revenue > 0)">
                    No info found
                </p>
            </div>
        </div>
    </div>
</template>
<script>
import { defineComponent, onMounted, ref, watch, computed } from 'vue';
import countrylist from '../assets/countrylist.js'
export default defineComponent({
    name: 'Movie',
    props: {
        idMovie: {
            type: Number,
            required: true,
        },
        apiKey: {
            type: String,
            required: true,
        }
    },

    emits: ['close'],

    setup: (props) => {
        const movie = ref()
        const countries = computed(() => {
            return Object.keys(movie.value["watch/providers"].results)
                .filter(countryName => movie.value["watch/providers"].results[countryName].flatrate)
                .map(countryCode => {
                    return {
                        code: countryCode,
                        name: countrylist.find(listItem => listItem.code === countryCode)?.name ?? countryCode
                    }
                });
        })
        const cast = computed(() => {
            return movie.value.credits.cast
                        .slice(0, 12)
        })
        const countryCode = ref("placeholder")


        const getMovieData = async () => {
            await fetch(`https://api.themoviedb.org/3/movie/${props.idMovie}?api_key=${props.apiKey}&append_to_response=watch/providers,credits`)
                .then(async res => res.json())
                .then(res => movie.value = res)
        }
        onMounted(async () => {
            await getMovieData()
        })

        watch(() => props.idMovie, getMovieData)

        return {
            movie,
            cast,
            countries,
            countryCode,
        }
    },
})
</script>
<style></style>
