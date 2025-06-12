<script setup>
import { ref, onMounted, onUnmounted } from "vue";
import BohrAtom from "./components/BohrAtom.vue";
import { gsap } from "gsap";
import { MotionPathPlugin } from "gsap/MotionPathPlugin";
gsap.registerPlugin(MotionPathPlugin);

const main = ref();
const atomRef = ref(null);
const ctx = gsap.context(() => {});

onMounted(() => {
  if (!atomRef.value) {
    console.warn("BohrAtom component not found");
    return;
  }
  ctx.add(() => {
    const orbitList = atomRef.value.orbitList;
    const electronList = atomRef.value.electronList;
    const orbit2 = orbitList[1];
    const electron = electronList[0];
    const atom = atomRef.value.rootRef;

    const electronTween = gsap.getTweensOf(electron)[0];

    if (!electronTween) {
      console.warn("Electron tween not found");
      return;
    }

    const timeline = gsap.timeline();

    timeline.to(atom, {
      x: 100,
      duration: 2,
      ease: "power1.inOut"
    });

    timeline.call(() => {
      electronTween.kill();
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
    const orbitPathRaw = MotionPathPlugin.getRawPath(targetOrbit);
    MotionPathPlugin.cacheRawPathMeasurements(orbitPathRaw);
    const p = MotionPathPlugin.getPositionOnPath(orbitPathRaw , 0.3);

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
