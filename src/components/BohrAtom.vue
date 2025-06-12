<template>
    <svg ref="rootRef" :width="size" :height="size" :viewBox="viewBox">
        <!-- Nucleus -->
        <circle v-for="(pos, index) in [...nucleusLayout].reverse()"
            :key="`nucleon-${index}`"
            :cx="pos[0]" :cy="pos[1]" :r="nucleonRadius"
            :class="index % 2 === 0 ? 'nucleon proton' : 'nucleon neutron'" />

        <!-- Orbitals -->
        <ellipse v-for="(electronCount, orbitIndex) in orbits"
            :key="`orbit-${orbitIndex}`"
            :cx="center" :cy="center"
            :rx="(1 + orbitIndex) * orbitRadius * orbitRadiusRatio[orbitIndex]"
            :ry="(1 + orbitIndex) * orbitRadius"
            :class="`orbit orbit-${orbitIndex}`" />

        <!-- Electrons -->
        <template v-for="(electronCount, orbitIndex) in orbits" :key="`electron-orbit-${orbitIndex}`">
            <circle v-for="i in electronCount"
                :key="`electron-${orbitIndex}-${i}`"
                :cx="center" :cy="center"
                :r="electronRadius"
                :class="`electron electron-orbit-${orbitIndex} electron-${i}`" />
        </template>
    </svg>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import { gsap } from 'gsap'
import { MotionPathPlugin } from 'gsap/MotionPathPlugin'

gsap.registerPlugin(MotionPathPlugin)

interface Props {
  orbits?: number[]
  speeds?: number[]
  orbitRadiusRatio?: number[]
  size?: number
}

const props = withDefaults(defineProps<Props>(), {
  orbits: () => [2, 3],
  speeds: () => [4, 2],
  orbitRadiusRatio: () => [1, 1.5],
  size: 200
})

// Destructure only the props used in the template
const { size, orbits, speeds, orbitRadiusRatio } = props
const center = size / 2
const viewBox = `0 0 ${size} ${size}`
const totalNucleons = 7 // Total number of nucleons (protons + neutrons)
const nucleonRadius = size / 50 // Radius of each nucleon
const electronRadius = size / 52 // Radius of the electron
const orbitRadius = size / 7 // Base radius for the first orbit
const nucleusLayout = generatePackedNucleusAlternating(totalNucleons, center, nucleonRadius)

const rootRef = ref<SVGSVGElement | null>(null)
let ctx!: ReturnType<typeof gsap.context>

defineExpose({
  rootRef,
  getOrbitRelativeDistance
})

onMounted(() => {
  if (!rootRef.value) {
    console.warn('BohrAtom is not yet set')
    return
  }
  console.log('BohrAtom mounted')

  ctx = gsap.context(() => {
    const orbits = gsap.utils.toArray<SVGEllipseElement>('.orbit')

    orbits.forEach((orbit, orbitIndex) => {
        const orbitPath = MotionPathPlugin.convertToPath(orbit)[0] as SVGPathElement
        const electrons_orbit = gsap.utils.toArray<SVGCircleElement>(`.electron-orbit-${orbitIndex}`)
        gsap.set(orbitPath, {
          rotation: 30 * orbitIndex,
          transformOrigin: 'center center'
        })

        electrons_orbit.forEach((electron, electronIndex) => {
          gsap.to(electron, {
            motionPath: {
                path: orbitPath,
                align: orbitPath,
                alignOrigin: [0.5, 0.5],
                start: electronIndex / electrons_orbit.length,
                end: (electronIndex / electrons_orbit.length) + 1,
            },
            duration: speeds[orbitIndex],
            repeat: -1,
            ease: 'linear'
          })
        })
    })

  }, rootRef.value)
})

onUnmounted(() => {
  ctx.revert()
})

function generatePackedNucleusAlternating(
  total: number,
  center: number,
  nucleonRadius: number,
  overlapFactor = 1.6,
): [number, number][] {
  if (total <= 0) {
    return []
  }

  const spacing = nucleonRadius * overlapFactor
  const positions: [number, number][] = [[center, center]]
  if (total === 1) {
    return positions
  }

  let count = 1
  let ring = 1

  while (count < total) {
    const radius = ring * spacing
    const steps = 6 * ring

    for (let i = 0; i < steps && count < total; i++) {
      const angle = (2 * Math.PI * i) / steps
      const x = center + radius * Math.cos(angle)
      const y = center + radius * Math.sin(angle)
      positions.push([x, y])
      count++
    }

    ring++
  }

  return positions
}

function getOrbitRelativeDistance(
  electronClass: string,
  targetOrbitClass: string,
  position: number
): { x: number; y: number } {
  const electron = gsap.utils.toArray<SVGCircleElement>(electronClass)[0]
  const targetOrbitElement = gsap.utils.toArray<SVGGeometryElement>(
    targetOrbitClass
  )[0]
  const targetOrbitPath = MotionPathPlugin.convertToPath(targetOrbitElement)[0]
  const orbitPathRaw = MotionPathPlugin.getRawPath(targetOrbitPath)
  MotionPathPlugin.cacheRawPathMeasurements(orbitPathRaw)
  const p = MotionPathPlugin.getPositionOnPath(orbitPathRaw, position)

  const bbox = targetOrbitPath.getBBox()
  const rl = MotionPathPlugin.getRelativePosition(
    electron,
    targetOrbitPath,
    [0.5, 0.5],
    {
      x: p.x - bbox.x,
      y: p.y - bbox.y
    }
  )

  return rl
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