<script setup>
import Controls from './Controls.vue'
import { ref, onMounted, onUnmounted } from 'vue'

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
const isListening = ref(false)
let recognition = null

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

  // Web Speech API for voice commands
  const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition
  if (SpeechRecognition) {
    recognition = new SpeechRecognition()
    recognition.continuous = true
    recognition.lang = 'tr-TR'
    recognition.interimResults = false

    recognition.onstart = () => {
      isListening.value = true
      console.log('Voice recognition started.')
    }

    recognition.onend = () => {
      isListening.value = false
      console.log('Voice recognition ended. Restarting...')
      // Restart recognition to keep it active
      if (recognition) {
        recognition.start()
      }
    }

    recognition.onerror = (event) => {
      console.error('Speech recognition error:', event.error)
    }

    recognition.onresult = (event) => {
      const last = event.results.length - 1
      const command = event.results[last][0].transcript.trim().toLowerCase()
      console.log('Command received:', command)
      handleVoiceCommand(command)
    }

    // Start listening
    recognition.start()
  } else {
    console.warn('Web Speech API is not supported in this browser.')
  }
})

onUnmounted(() => {
  if (recognition) {
    recognition.stop()
    recognition = null
  }
})

const handleVoiceCommand = (command) => {
  const commandMap = {
    'oynat': () => {
      if (!isPlaying.value) videoElement.value.play()
    },
    'duraklat': () => {
      if (isPlaying.value) videoElement.value.pause()
    },
    'başa sar': () => {
      seek(0)
    },
    'ileri sar': () => {
      seek(Math.min(duration.value, currentTime.value + 5))
    },
    'geri sar': () => {
      seek(Math.max(0, currentTime.value - 5))
    }
  }

  const action = commandMap[command]

  if (action) {
    action()
  } else {
    console.log('Unknown command:', command)
  }
}

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
