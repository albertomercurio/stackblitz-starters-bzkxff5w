<script setup>
import { ref, onMounted, onUnmounted } from "vue";
import BohrAtom from "./components/BohrAtom.vue";
import { gsap } from "gsap";
import { MotionPathPlugin } from "gsap/MotionPathPlugin";
gsap.registerPlugin(MotionPathPlugin);

const main = ref();
const atomRef = ref();

</script>

<template>
  <main>
    <section class="boxes-container" ref="main">
      <BohrAtom ref="atomRef" />
    </section>
  </main>
</template>
