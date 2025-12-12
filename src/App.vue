<script setup>
import LatencyMonitor from './components/LatencyMonitor.vue'
</script>

<template>
  <LatencyMonitor />
</template>

<style>
/* Global reset for the app to ensure full screen usage */
body {
  margin: 0;
  padding: 0;
  background: #0f172a;
}
#app {
  width: 100%;
  max-width: 100%;
  margin: 0;
  padding: 0;
}
</style>
