<template>
  <v-layout class="rounded rounded-md">

    <v-main
      class="d-flex align-center justify-center"
      style="min-height: 300px">
      <router-view />
    </v-main>
  </v-layout>
</template>

<script setup>
  import { ref } from 'vue';
  const drawer = ref(false);
</script>
