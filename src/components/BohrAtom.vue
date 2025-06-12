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
const ctx = gsap.context(() => {});

defineExpose({
    rootRef,
    getOrbitRelativeDistance,
})

onMounted(() => {
    if (!rootRef.value) {
        console.warn('BohrAtom is not yet set');
        return;
    }
    console.log('BohrAtom mounted');

    ctx.add(() => {
        const orbits =  gsap.utils.toArray('.orbit');
        const electron = gsap.utils.toArray('.electron')[0];
        const orbitPaths = orbits.map(orbit => MotionPathPlugin.convertToPath(orbit)[0]);
        const orbitPath = orbitPaths[0]; // Use the first orbit path for the electron

        gsap.set(orbitPaths, {
            rotation: 45,
            transformOrigin: 'center center'
        });

        gsap.to(electron, {
            motionPath: {
                path: orbitPath,
                align: orbitPath,
                alignOrigin: [0.5, 0.5],
                start: 0,
                end: 1,
            },
            duration: 2,
            repeat: -1,
            ease: "linear",
        });
    }, rootRef.value);
})

onUnmounted(() => {
    ctx.revert(); // Cleanup GSAP context
});

function getOrbitRelativeDistance(electronClass, targetOrbitClass) {
    const electron = gsap.utils.toArray(electronClass)[0];
    const targetOrbitPath = MotionPathPlugin.convertToPath(targetOrbitClass)[0];
    const orbitPathRaw = MotionPathPlugin.getRawPath(targetOrbitPath);
    MotionPathPlugin.cacheRawPathMeasurements(orbitPathRaw);
    const p = MotionPathPlugin.getPositionOnPath(orbitPathRaw , 0.8);

    const bbox = targetOrbitPath.getBBox();
    const rl = MotionPathPlugin.getRelativePosition(electron, targetOrbitPath, [0.5, 0.5], {
        x: p.x - bbox.x,
        y: p.y - bbox.y
    });

    return rl;
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