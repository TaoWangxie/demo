<template>
  <div class="container">ceshi 2223  </div>
</template>

<script setup lang='ts'>
import { nextTick } from 'vue'

nextTick(() => {
  document.documentElement.classList.add('htmlClass')
})
</script>

<style scoped>
.container {
  width: 100%;
  height: 100vh;
  display: flex;
  overflow-x: hidden;
}
</style>
