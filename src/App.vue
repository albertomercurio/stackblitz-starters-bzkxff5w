<script setup>
import { onMounted, onUnmounted, ref } from "vue";
import gsap from "gsap";
import BohrAtom from "./components/BohrAtom.vue";

const main = ref();
const tl = ref();
const ctx = ref();

const toggleTimeline = () => {
  tl.value.reversed(!tl.value.reversed());
};
// onMounted(() => {
//   ctx.value = gsap.context((self) => {
//     const boxes = self.selector(".box");
//     tl.value = gsap.timeline()
//       .to(boxes[0], { x: 120, rotation: 360 })
//       .to(boxes[1], { x: -120, rotation: -360 }, "<")
//       .to(boxes[2], { y: -166 })
//       .reverse();
//   }, main.value) // <- Scope!
// });

onUnmounted(() => {
  ctx.value.revert(); // <- Easy Cleanup!
});
</script>

<template>
  <main>
    <section class="boxes-container" ref="main">
      <BohrAtom />
    </section>
  </main>
</template>
