<template>
    <svg ref="rootRef" width=200 height=200 viewBox="0 0 200 200">
        <circle cx=100 cy=100 r=5 class="nucleon proton" />

        <ellipse cx=100 cy=100 rx=20 ry=20 class="orbit orbit1" />
        <ellipse cx=100 cy=100 rx=60 ry=40 class="orbit orbit2" />

        <circle cx=100 cy=100 r=4 class="electron" />
    </svg>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import { gsap } from 'gsap'
import { MotionPathPlugin } from 'gsap/MotionPathPlugin'

gsap.registerPlugin(MotionPathPlugin)

const rootRef = ref(null);
const ctx = ref(null);

onMounted(() => {
    console.log('BohrAtom mounted');

    ctx.value = gsap.context(() => {
        const orbit1 = document.querySelector('.orbit1')
        const orbit2 = document.querySelector('.orbit2')
        const electron = document.querySelector('.electron')

        gsap.set(orbit2, {
            rotation: 45,
            transformOrigin: 'center center'
        })

        gsap.to(electron, {
            attr: {
                cx: getOrbitRelativeDistance(electron, orbit2).x,
                cy: getOrbitRelativeDistance(electron, orbit2).y
            },
            duration: 2,
        })
    }, rootRef.value);
})

onUnmounted(() => {
    ctx.value?.revert(); // Cleanup GSAP context
})


function getOrbitRelativeDistance(electron, targetOrbit) {
    const targetOrbitPath = MotionPathPlugin.convertToPath(targetOrbit)[0];
    const orbitPathRaw = MotionPathPlugin.getRawPath(targetOrbitPath);
    MotionPathPlugin.cacheRawPathMeasurements(orbitPathRaw);
    const p = MotionPathPlugin.getPositionOnPath(orbitPathRaw , 0.3);

    console.log('Position on path:', p);

    return p;
}
</script>

<style scoped>
.electron {
    fill: #1e90ff;
}

.nucleon {
    stroke: black;
    stroke-width: 0.5;
}

.proton {
    fill: #dc3e2c;
}

.orbit {
    fill: none;
    stroke: #888;
    stroke-dasharray: 4 4;
}
</style>