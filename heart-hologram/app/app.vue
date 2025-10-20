<template>
  <div>
    <!-- Trái tim hologram chính -->
    <Heart3D v-if="!showMessage" @clicked="showMessage = true" />

    <!-- Thông điệp -->
    <div v-if="showMessage" class="message">
      Chúc em có ngày 20-10 hạnh phúc cục cưng bé bỏng của anh
    </div>

    <!-- Trái tim nhỏ bay quanh -->
    <span
      v-for="(h, index) in hearts"
      :key="index"
      class="small-heart"
      :style="{
        left: h.x + 'vw',
        animationDuration: h.duration + 's',
        fontSize: h.size + 'px'
      }"
    >
      💙
    </span>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import Heart3D from '~/components/Heart3D.vue'

const showMessage = ref(false)

// Tạo nhiều trái tim nhỏ ngẫu nhiên
const hearts = Array.from({ length: 40 }).map(() => ({
  x: Math.random() * 100,
  duration: 5 + Math.random() * 8,
  size: 12 + Math.random() * 20
}))
</script>

<style>
@import "@/assets/css/main.css";
</style>
