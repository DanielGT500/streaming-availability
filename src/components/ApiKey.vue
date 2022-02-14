<template>
    <div class="api-card">
        <div class="input-card">
            <input type="text" class="api-input-field" placeholder="Please enter your TMDB API Key" v-model="apiInput" @keyup.enter="saveApiKey">
            <button class="api-sender" @click="saveApiKey">Submit Key</button>
        </div>
    </div>
</template>
<script>
import { defineComponent, ref } from 'vue';

export default defineComponent({
    name: 'ApiKey',
    emits: ['apiInput'],
    setup: (props, context) => {
        const apiInput = ref('')
        const saveApiKey = () => {
            localStorage.setItem(`apiKey`, `${apiInput.value}`)
            context.emit('apiInput', apiInput.value)
        }

        return {
            apiInput,
            saveApiKey,
        }
    }
})
</script>
<style>
</style>