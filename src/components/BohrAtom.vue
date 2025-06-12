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
const orbitList = ref([]);
const electronList = ref([]);
const ctx = gsap.context(() => {});

defineExpose({
    rootRef,
    orbitList,
    electronList,
})

onMounted(() => {
    if (!rootRef.value) {
        console.warn('BohrAtom is not yet set');
        return;
    }
    console.log('BohrAtom mounted');

    ctx.add(() => {
        const orbits =  gsap.utils.toArray('.orbit')
        const electrons = gsap.utils.toArray('.electron');
        const electron = electrons[0];
        const orbitPaths = orbits.map(orbit => MotionPathPlugin.convertToPath(orbit)[0]);
        const orbitPath = orbitPaths[0]; // Use the first orbit path for the electron

        gsap.set(orbitPaths, {
            rotation: 45,
            transformOrigin: 'center center'
        });

        const timeline = gsap.timeline();
        const atom = rootRef.value;

        console.log('Electron:', electron);

        timeline.to(electron, {
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
                cx: () => "+=" + getOrbitRelativeDistance(electron, orbitPaths[1]).x,
                cy: () => "+=" + getOrbitRelativeDistance(electron, orbitPaths[1]).y
            },
            duration: 2,
        });

        console.log('Electron Post:', gsap.utils.toArray('.electron')[0]);

        orbitList.value = gsap.utils.toArray('.orbit');
        electronList.value = gsap.utils.toArray('.electron');

    }, rootRef.value);
})

onUnmounted(() => {
    ctx.revert(); // Cleanup GSAP context
})

// function getOrbitRelativeDistance(electron, targetOrbit) {
//     const orbitPathRaw = MotionPathPlugin.getRawPath(targetOrbit);
//     MotionPathPlugin.cacheRawPathMeasurements(orbitPathRaw);
//     const p = MotionPathPlugin.getPositionOnPath(orbitPathRaw , 0.3);

//     console.log('Path Position:', p);

//     return p;
// }

function getOrbitRelativeDistance(electron0, targetOrbit0) {
    const electron = gsap.utils.toArray(".electron")[0];
    const targetOrbit = gsap.utils.toArray(".orbit")[1];
    const targetOrbitPath = MotionPathPlugin.convertToPath(targetOrbit)[0];
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