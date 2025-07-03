<script setup>
import Player from './components/video-player/Player.vue'
</script>

<template>
  <Player :videoUrl="'https://www.w3schools.com/html/mov_bbb.mp4'" />

  <footer>
    <small>
      <a href="https://github.com/gayret">Safa Gayret</a> tarafından geliştirildi.
      <br />
      Kodlara <a href="https://github.com/gayret/vue-speech-recognition">GitHub</a>'dan ulaşılabilir.
    </small>

  </footer>
</template>

<style scoped>
a {
  text-decoration: none;
}

footer {
  margin-top: 3rem;
  color: #666;
  text-align: center;
}
</style>