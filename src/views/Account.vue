<template>
    <main>
        <sign-in v-if="signIn" @switch="signIn = !signIn" />
        <sign-up v-else @switch="signIn = !signIn" />
    </main>
</template>

<script setup lang="ts">
import SignIn from '@/components/modals/SignIn.vue'
import SignUp from '@/components/modals/SignUp.vue'

import { ref } from 'vue'
let signIn = ref<boolean>(false)
</script>

<style scoped>
main {
    width: 100vw;
    height: 100vh;
    overflow: hidden;
}
</style>