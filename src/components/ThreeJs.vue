<template>
  <div ref="content"></div>
</template>

<script setup>
import * as THREE from 'three'
import { onMounted, ref } from 'vue'

const content = ref()
function create() {
  const renderer = new THREE.WebGLRenderer()
  renderer.setSize(400, 400)
  content.value.appendChild(renderer.domElement)

  const camera = new THREE.PerspectiveCamera(
    20,
    400 / 400,
    1,
    500
  )
  camera.position.set(30, 70, 100)
  camera.lookAt(0, 0, 0)

  const scene = new THREE.Scene()
  const material = new THREE.LineBasicMaterial({ color: 0x0000ff })
  const points = []
  points.push(new THREE.Vector3(-10, 0, 0))
  points.push(new THREE.Vector3(0, 10, 0))
  points.push(new THREE.Vector3(10, 0, 0))

  const geometry = new THREE.BufferGeometry().setFromPoints(points)
  const line = new THREE.Line(geometry, material)
  scene.add(line)
  renderer.render(scene, camera)
}

onMounted(() => {
  create()
})
</script>

