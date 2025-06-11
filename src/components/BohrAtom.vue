<template>
    <svg ref="rootRef" :width="size" :height="size" :viewBox="viewBox">
        <!-- Nucleus -->
         <circle :cx="center" :cy="center" :r="nucleonRadius" class="nucleon proton" />

        <!-- Orbitals -->
        <ellipse :cx="center" :cy="center" :rx="radiusStep" :ry="radiusStep" :transform="`rotate(45 ${center} ${center})`" class="orbit" />
        <ellipse :cx="center" :cy="center" :rx="2*1.5*radiusStep" :ry="2*radiusStep" :transform="`rotate(45 ${center} ${center})`" class="orbit" />

        <!-- Electron -->
        <circle :cx="center" :cy="center" r="4" class="electron" />
    </svg>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import { gsap } from 'gsap'
import { MotionPathPlugin } from 'gsap/MotionPathPlugin'

gsap.registerPlugin(MotionPathPlugin)

const props = defineProps({
    size: { type: Number, default: 400 },
})

const center = props.size / 2
const radiusStep = props.size / 8
const viewBox = `0 0 ${props.size} ${props.size}`
const nucleonRadius = 4

const rootRef = ref(null);
const orbitsList = ref([]);
const electronsList = ref([[]]);
const ctx = ref(null);

defineExpose({
    rootRef,
    orbitsList,
    electronsList,
})

onMounted(() => {
    console.log('BohrAtom mounted');
    populateOrbitsList(rootRef.value);
    populateElectronsList(rootRef.value, orbitsList.value);
    startAnimation(orbitsList.value, electronsList.value);
})

onUnmounted(() => {
    ctx.value?.revert(); // Cleanup GSAP context
})

function startAnimation(orbitsList, electronsList) {
    ctx.value = gsap.context(() => {
        const [orbitPath] = MotionPathPlugin.convertToPath(orbitsList[0]);
        gsap.to(electronsList[0][0], {
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
    });
}

// Populate the orbits and electrons lists based on the SVG structure at runtime
// This avoids possible conflicts
function populateOrbitsList(rootRef) {
    orbitsList.value = Array.from(rootRef.querySelectorAll('.orbit'));
}

function populateElectronsList(rootRef, orbitsList) {
    orbitsList.forEach((_, orbitIndex) => {
        const electrons = rootRef.querySelectorAll(`.electron`);
        electronsList.value[orbitIndex] = Array.from(electrons);
    });
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