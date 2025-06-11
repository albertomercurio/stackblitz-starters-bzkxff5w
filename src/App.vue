<script setup>
import { onMounted, onUnmounted, ref } from "vue";
import BohrAtom from "./components/BohrAtom.vue";

const main = ref();
const atomRef = ref(null);

</script>

<template>
  <main>
    <section class="boxes-container" ref="main">
      <BohrAtom ref="atomRef" />
    </section>
  </main>
</template>
