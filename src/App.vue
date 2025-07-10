<script setup lang="ts">
import FileUpload from './components/FileUpload.vue'
</script>

<template>
  <div class="app-container">
    <FileUpload />
  </div>
</template>

<style>
.app-container {
  min-height: 100vh;
  background: #f5f5f5;
  padding: 0;
}

@media (prefers-color-scheme: dark) {
  .app-container {
    background: #1a1a1a;
  }
}
</style>
