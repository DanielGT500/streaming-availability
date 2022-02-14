<template>
    <div class="card" @click="$emit('close')">
        <div class="movie-cols" @click.stop="" v-if="tv">
            <div class="poster">
                <img v-if="tv.poster_path" class="movie-poster" :src="`https://image.tmdb.org/t/p/w500${tv.poster_path}`"/>
                <div v-else>
                    <img class="no-poster-img" src="../assets/no-poster.png" />
                    <p class="card-no-poster-title">{{ tv.name }}</p>
                </div>
            </div>
            <div class="movie-info">
                <div class="link-container">
                    <a v-if="tv.homepage" class="links website" :href="`${tv.homepage}`" target="blank"><span>Official Website</span></a>
                </div>
                <p class="provider" v-if="countries.length">
                    <select v-model="countryCode" name="countries" class="countries" required>
                        <option value="placeholder" disabled selected hidden>Streaming on</option>
                        <option v-for="(country, index) in countries" :value="country.code" :key="index">{{ country.name }}</option>
                    </select><br>
                    {{ tv["watch/providers"].results[countryCode]?.flatrate.map(x => x.provider_name).join(', ') }}
                </p>
                <p v-if="tv.genres.length"><span>Genre(s)</span><br> {{ tv.genres.map(x => x.name).join('/') }}</p>
                <p v-if="tv.first_air_date"><span>First air date</span><br> {{ tv.first_air_date }}</p>
                <p v-if="tv.last_air_date"><span>Last air date</span><br> {{ tv.last_air_date }}</p>
                <p v-if="tv.number_of_seasons"><span>{{ tv.number_of_seasons }} Season(s)</span></p>
                <p v-if="tv.number_of_episodes"><span>{{ tv.number_of_episodes }} Episode(s)</span></p>
                <p v-if="tv.episode_run_time && (tv.episode_run_time[0] > 0)"><span>Episode runtime</span><br> {{ tv.episode_run_time[0] }} minutes</p>
                <p v-if="tv.overview"><span>Overview</span><br> {{ tv.overview}} </p>
            </div>
        </div>
    </div>
</template>
<script>
import { defineComponent, onMounted, ref, watch, computed } from 'vue';
import countrylist from '../assets/countrylist.js'

export default defineComponent({
    name: 'Tv',
    props: {
        idTv: {
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
        const tv = ref()
        const countries = computed(() => {
            return Object.keys(tv.value["watch/providers"].results)
                .filter(countryName => tv.value["watch/providers"].results[countryName].flatrate)
                .map(countryCode => {
                    return {
                        code: countryCode,
                        name: countrylist.find(listItem => listItem.code === countryCode)?.name ?? countryCode
                    }
                });
        })

        const countryCode = ref("placeholder")

        const getTvData = async () => {
            await fetch(`https://api.themoviedb.org/3/tv/${props.idTv}?api_key=${props.apiKey}&append_to_response=watch/providers`)
                .then(async res => res.json())
                .then(res => tv.value = res)
        }
        onMounted(async () => {
            await getTvData()
        })

        watch(() => props.id, getTvData)

        return {
            tv,
            countries,
            countryCode,
        }
    },
})
</script>
<style></style>