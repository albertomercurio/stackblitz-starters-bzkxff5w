<script setup>
import { onMounted, onUnmounted, ref } from "vue";
import BohrAtom from "./components/BohrAtom.vue";
import { gsap } from "gsap";
import { MotionPathPlugin } from "gsap/MotionPathPlugin";
gsap.registerPlugin(MotionPathPlugin);

const main = ref();
const atomRef = ref(null);
const ctx = gsap.context(() => {});

onMounted(() => {
  ctx.add(() => {
    const orbitList = atomRef.value.orbitList;
    const electronList = atomRef.value.electronList;
    const orbit2 = orbitList[1];
    const electron = electronList[0];

    const timeline = gsap.timeline();
    timeline.to(atomRef.value.rootRef, {
      x: 100,
      duration: 2,
      ease: "power1.inOut"
    });

    timeline.to(electron, {
        attr: {
            cx: getOrbitRelativeDistance(electron, orbit2).x,
            cy: getOrbitRelativeDistance(electron, orbit2).y
        },
        duration: 2,
    });
  }, main.value);
});

onUnmounted(() => {
  ctx.revert(); // Cleanup GSAP context
});

function getOrbitRelativeDistance(electron, targetOrbit) {
    const targetOrbitPath = MotionPathPlugin.convertToPath(targetOrbit)[0];
    const orbitPathRaw = MotionPathPlugin.getRawPath(targetOrbitPath);
    MotionPathPlugin.cacheRawPathMeasurements(orbitPathRaw);
    const p = MotionPathPlugin.getPositionOnPath(orbitPathRaw , 0.3);

    console.log('Position on path:', p);

    return p;
}

</script>

<template>
  <main>
    <section class="boxes-container" ref="main">
      <BohrAtom ref="atomRef" />
    </section>
  </main>
</template>
