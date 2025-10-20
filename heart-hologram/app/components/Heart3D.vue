<template>
  <div ref="container" class="three-container">
    <!-- Scanline overlay -->
    <div class="scanline-overlay"></div>
    <!-- Glitch overlay -->
    <div class="glitch-overlay"></div>
    <!-- Hiệu ứng bóng màu -->
    <div class="chromatic-ghost chroma-red"></div>
    <div class="chromatic-ghost chroma-cyan"></div>
  </div>
</template>

<script setup>
import { onMounted, onBeforeUnmount, ref } from 'vue'
import * as THREE from 'three'

const container = ref(null)
const emit = defineEmits(['clicked'])

let renderer, scene, camera, heart, particles, animationId
let glitchTimer = 0
let isGlitching = false

onMounted(() => {
  // Scene và camera
  scene = new THREE.Scene()
  camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 1000)
  camera.position.set(0, 0, 40)
  camera.lookAt(0, 0, 0)

  // Renderer
  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
  container.value.appendChild(renderer.domElement)

  // Hình trái tim
  const x = 0, y = 0
  const heartShape = new THREE.Shape()
  heartShape.moveTo(x + 5, y + 5)
  heartShape.bezierCurveTo(x + 5, y + 5, x + 4, y, x, y)
  heartShape.bezierCurveTo(x - 6, y, x - 6, y + 7, x - 6, y + 7)
  heartShape.bezierCurveTo(x - 6, y + 11, x - 3, y + 15.4, x + 5, y + 19)
  heartShape.bezierCurveTo(x + 12, y + 15.4, x + 16, y + 11, x + 16, y + 7)
  heartShape.bezierCurveTo(x + 16, y + 7, x + 16, y, x + 10, y)
  heartShape.bezierCurveTo(x + 7, y, x + 5, y + 5, x + 5, y + 5)

  const geometry = new THREE.ExtrudeGeometry(heartShape, {
    depth: 3,
    bevelEnabled: true,
    bevelSegments: 2,
    steps: 2,
    bevelSize: 1.2,
    bevelThickness: 1.2
  })
  geometry.center()

  const material = new THREE.MeshPhongMaterial({
    color: 0x00eaff,
    emissive: 0x0077cc,
    transparent: true,
    opacity: 0.75,
    wireframe: true
  })

  heart = new THREE.Mesh(geometry, material)
  heart.rotation.z = Math.PI; // Xoay 180 độ quanh trục Z để trái tim đúng chiều
  scene.add(heart)

  // Hạt sáng bên trong
  const particlesGeometry = new THREE.BufferGeometry()
  const particlesCount = 400
  const positions = new Float32Array(particlesCount * 3)
  for (let i = 0; i < particlesCount; i++) {
    positions[i * 3] = (Math.random() - 0.5) * 24
    positions[i * 3 + 1] = (Math.random() - 0.5) * 24
    positions[i * 3 + 2] = (Math.random() - 0.5) * 8
  }
  particlesGeometry.setAttribute('position', new THREE.Float32BufferAttribute(positions, 3))
  const particlesMaterial = new THREE.PointsMaterial({
    color: 0x00eaff,
    size: 0.25,
    transparent: true,
    opacity: 0.9
  })
  particles = new THREE.Points(particlesGeometry, particlesMaterial)
  scene.add(particles)

  // Ánh sáng
  scene.add(new THREE.AmbientLight(0x226688, 0.6))
  const point1 = new THREE.PointLight(0x00eaff, 1.5, 200)
  point1.position.set(20, 20, 30)
  scene.add(point1)
  const point2 = new THREE.PointLight(0x00bfff, 1.2, 200)
  point2.position.set(-20, -10, -20)
  scene.add(point2)

  // Animation
  const clock = new THREE.Clock()
  const animate = () => {
    animationId = requestAnimationFrame(animate)
    const t = clock.getElapsedTime()

    heart.rotation.y = t * 0.25
    heart.rotation.x = Math.sin(t * 0.2) * 0.1

    particles.rotation.y = -t * 0.1
    particles.rotation.x = Math.sin(t * 0.15) * 0.05

    const s = 1 + Math.sin(t * 1.5) * 0.03
    heart.scale.set(s, s, s)

    glitchTimer += clock.getDelta()
    if (!isGlitching && glitchTimer > 3 + Math.random() * 4) {
      isGlitching = true
      glitchTimer = 0
      startGlitch(300)
    }

    renderer.render(scene, camera)
  }
  animate()

  // Click → emit
  renderer.domElement.addEventListener('click', () => {
    emit('clicked')
  })

  // Resize
  const onResize = () => {
    camera.aspect = window.innerWidth / window.innerHeight
    camera.updateProjectionMatrix()
    renderer.setSize(window.innerWidth, window.innerHeight)
  }
  window.addEventListener('resize', onResize)
})

function startGlitch(duration = 300) {
  const rootEl = container.value
  if (!rootEl) return
  rootEl.classList.add('glitching')

  const jitterStart = performance.now()
  const jitter = () => {
    if (performance.now() - jitterStart < duration) {
      const jx = (Math.random() - 0.5) * 0.08
      const jy = (Math.random() - 0.5) * 0.08
      heart.rotation.y += jx
      heart.rotation.x += jy
      camera.position.x = jx * 10
      camera.position.y = jy * 10
      requestAnimationFrame(jitter)
    } else {
      camera.position.x = 0
      camera.position.y = 0
      container.value?.classList.remove('glitching')
      isGlitching = false
    }
  }
  jitter()
}

onBeforeUnmount(() => {
  cancelAnimationFrame(animationId)
  if (renderer) renderer.dispose()
})
</script>

<style>
.three-container {
  width: 100vw;
  height: 100vh;
  background: black;
  overflow: hidden;
  position: relative;
}

/* Khi glitch, container rung nhẹ */
.three-container.glitching {
  animation: containerJitter 0.2s steps(2, end) infinite;
}

@keyframes containerJitter {
  0% { transform: translate(0, 0); }
  50% { transform: translate(-1px, 1px); }
  100% { transform: translate(0, 0); }
}
</style>
