<script setup>
import Controls from './Controls.vue'
import { ref, onMounted } from 'vue'

const props = defineProps({
  videoUrl: {
    type: String,
    required: true
  }
})

const videoElement = ref(null)
const isPlaying = ref(false)
const currentTime = ref(0)
const duration = ref(0)
const volume = ref(1)
const isMuted = ref(false)

onMounted(() => {
  if (videoElement.value) {
    // Set initial values
    const setInitialValues = () => {
      duration.value = videoElement.value.duration
      volume.value = videoElement.value.volume
      isMuted.value = videoElement.value.muted
    }

    // Add event listeners
    videoElement.value.addEventListener('loadedmetadata', setInitialValues)
    videoElement.value.addEventListener('durationchange', setInitialValues) // Handle case where duration changes

    videoElement.value.addEventListener('timeupdate', () => {
      currentTime.value = videoElement.value.currentTime
    })

    videoElement.value.addEventListener('play', () => {
      isPlaying.value = true
    })

    videoElement.value.addEventListener('pause', () => {
      isPlaying.value = false
    })

    videoElement.value.addEventListener('ended', () => {
      isPlaying.value = false
    })

    videoElement.value.addEventListener('volumechange', () => {
      volume.value = videoElement.value.volume
      isMuted.value = videoElement.value.muted
    })

    // If metadata is already loaded
    if (videoElement.value.readyState >= 1) {
      setInitialValues()
    }
  }
})

const togglePlayPause = () => {
  if (isPlaying.value) {
    videoElement.value.pause()
  } else {
    videoElement.value.play()
  }
}

const seek = (time) => {
  if (videoElement.value) {
    videoElement.value.currentTime = time
  }
}

const updateVolume = (newVolume) => {
  if (videoElement.value) {
    videoElement.value.volume = newVolume
  }
}

const toggleMute = () => {
  if (videoElement.value) {
    videoElement.value.muted = !videoElement.value.muted
  }
}

const toggleFullscreen = () => {
  if (videoElement.value.requestFullscreen) {
    videoElement.value.requestFullscreen()
  } else if (videoElement.value.mozRequestFullScreen) { /* Firefox */
    videoElement.value.mozRequestFullScreen()
  } else if (videoElement.value.webkitRequestFullscreen) { /* Chrome, Safari & Opera */
    videoElement.value.webkitRequestFullscreen()
  } else if (videoElement.value.msRequestFullscreen) { /* IE/Edge */
    videoElement.value.msRequestFullscreen()
  }
}

</script>

<template>
  <div class="video-player">
    <video ref="videoElement" width="500" @dblclick="toggleFullscreen">
      <source :src="props.videoUrl" type="video/mp4">
    </video>

    <Controls :isPlaying="isPlaying" :currentTime="currentTime" :duration="duration" :volume="volume" :isMuted="isMuted"
      @togglePlayPause="togglePlayPause" @seek="seek" @updateVolume="updateVolume" @toggleMute="toggleMute"
      @toggleFullscreen="toggleFullscreen" />
  </div>
</template>

<style scoped>
.video-player {
  position: relative;
  width: 500px;
}
</style>
