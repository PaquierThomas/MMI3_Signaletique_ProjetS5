<script setup>
import { RouterLink, RouterView } from "vue-router";
import MapOsm from '../src/components/MapOsm.vue';
</script>

<template>
  <header>
    <nav>
      <RouterLink class="px-2" to="/">Home</RouterLink>
      <RouterLink to="/about">About</RouterLink>
      <div class="container-fluid">
        <!-- <h1> Open Street Map - Carte</h1> -->
        <MapOsm/>
    </div>
    </nav>
  </header>

  <RouterView />
</template>

