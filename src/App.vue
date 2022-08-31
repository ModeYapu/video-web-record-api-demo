<script setup>
import videoVue from './components/picurte.vue';
import video1Vue from './components/video1.vue';
import ffmpegVue from './components/ffmpeg.vue';
import pageVideo from './pages/video.vue'
</script>

<template>
  <!-- <videoVue /> -->
  <!-- <video1Vue></video1Vue> -->
  <!-- <ffmpegVue></ffmpegVue> -->
  <pageVideo></pageVideo>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
