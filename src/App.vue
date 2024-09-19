<script setup lang="ts">
import { ref, onMounted } from 'vue';
import Loading from './components/loading.vue';
import Camera from './components/camera.vue'
const loaded = ref(false);

onMounted(() => {
  setTimeout(() => {
    loaded.value = true;
  }, 20000); 
});
</script>

<template>
  <section class="w-screen  h-screen items-center justify-center flex flex-col">
    <Loading v-if="!loaded"/>
    <Camera v-if="loaded"/>
  </section>
</template>

<style scoped>
/* Optional: Add component-specific styles here */
</style>
