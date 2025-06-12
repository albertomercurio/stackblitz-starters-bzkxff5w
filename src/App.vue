<script setup>
import { ref, onMounted, onUnmounted } from "vue";
import BohrAtom from "./components/BohrAtom.vue";
import { gsap } from "gsap";
import { MotionPathPlugin } from "gsap/MotionPathPlugin";
gsap.registerPlugin(MotionPathPlugin);

const main = ref();
const atomRef = ref();
const ctx = gsap.context(() => {});

onMounted(() => {
  ctx.add(() => {
    const atom = atomRef.value.rootRef;
    const electron = gsap.utils.toArray(".electron")[0];

    const timeline = gsap.timeline();

    timeline.to(atom, {
        x: 100,
        duration: 2,
        ease: "power1.inOut"
    });

    timeline.call(() => {
        let electronTween = gsap.getTweensOf(".electron")[0];
        electronTween.pause();
    });

    timeline.to(electron, {
        attr: {
            cx: () => "+=" + atomRef.value.getOrbitRelativeDistance(".electron", ".orbit2").x,
            cy: () => "+=" + atomRef.value.getOrbitRelativeDistance(".electron", ".orbit2").y
        },
        duration: 2,
    });

  }, main.value);
});

onUnmounted(() => {
  ctx.revert();
});

</script>

<template>
  <main>
    <section class="boxes-container" ref="main">
      <BohrAtom ref="atomRef" />
    </section>
  </main>
</template>
