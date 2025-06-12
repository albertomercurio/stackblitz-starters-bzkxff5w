<template>
    <svg ref="rootRef" width=200 height=200 viewBox="0 0 200 200">
        <circle cx=100 cy=100 r=5 class="nucleon proton" />

        <ellipse cx=100 cy=100 rx=20 ry=20 class="orbit orbit1" />
        <ellipse cx=100 cy=100 rx=60 ry=40 class="orbit orbit2" />

        <circle cx=100 cy=100 r=4 class="electron" />
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
const { size } = props
const center = size / 2

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
    const electron = gsap.utils.toArray<SVGCircleElement>('.electron')[0]
    const orbitPaths = orbits.map(orbit =>
      (MotionPathPlugin.convertToPath(orbit)[0] as SVGPathElement)
    )
    const orbitPath = orbitPaths[0]

    gsap.set(orbitPaths, {
      rotation: 30,
      transformOrigin: 'center center'
    })

    gsap.to(electron, {
      motionPath: {
        path: orbitPath,
        align: orbitPath,
        alignOrigin: [0.5, 0.5],
        start: 0,
        end: 1
      },
      duration: 2,
      repeat: -1,
      ease: 'linear'
    })
  }, rootRef.value)
})

onUnmounted(() => {
  ctx.revert()
})

function getOrbitRelativeDistance(
  electronClass: string,
  targetOrbitClass: string
): { x: number; y: number } {
  const electron = gsap.utils.toArray<SVGCircleElement>(electronClass)[0]
  const targetOrbitElement = gsap.utils.toArray<SVGGeometryElement>(
    targetOrbitClass
  )[0]
  const targetOrbitPath = MotionPathPlugin.convertToPath(targetOrbitElement)[0]
  const orbitPathRaw = MotionPathPlugin.getRawPath(targetOrbitPath)
  MotionPathPlugin.cacheRawPathMeasurements(orbitPathRaw)
  const p = MotionPathPlugin.getPositionOnPath(orbitPathRaw, 0.8)

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