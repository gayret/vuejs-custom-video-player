<script setup>
import VolumeSlider from './VolumeSlider.vue'

const props = defineProps({
  isPlaying: Boolean,
  currentTime: Number,
  duration: Number,
  volume: Number,
  isMuted: Boolean
})

const emit = defineEmits([
  'togglePlayPause',
  'seek',
  'updateVolume',
  'toggleMute',
  'toggleFullscreen'
])

const formatTime = (timeInSeconds) => {
  const hours = Math.floor(timeInSeconds / 3600)
  const minutes = Math.floor((timeInSeconds % 3600) / 60)
  const seconds = Math.floor(timeInSeconds % 60)

  return [
    hours > 0 ? hours.toString().padStart(2, '0') : null,
    minutes.toString().padStart(2, '0'),
    seconds.toString().padStart(2, '0')
  ].filter(Boolean).join(':')
}

const voiceCommands = [
  'Oynat',
  'Duraklat',
  'Başa Sar',
  'İleri Sar',
  'Geri Sar'
]
</script>

<template>
  <div class="controls-container">
    <input type="range" min="0" :max="duration || 0" :value="currentTime" step="1"
      @input="emit('seek', parseFloat($event.target.value))" class="progress-bar" />
    <div class="bottom-controls">
      <div class="left-controls">
        <button @click="emit('togglePlayPause')" class="icon-button">
          <svg v-if="!isPlaying" viewBox="0 0 24 24">
            <path d="M8 5v14l11-7z" />
          </svg>
          <svg v-else viewBox="0 0 24 24">
            <path d="M6 19h4V5H6v14zm8-14v14h4V5h-4z" />
          </svg>
        </button>
        <VolumeSlider :volume="volume" :isMuted="isMuted" @updateVolume="emit('updateVolume', $event)"
          @toggleMute="emit('toggleMute')" />
        <div class="time-display">
          <span>{{ formatTime(currentTime) }} / {{ formatTime(duration || 0) }}</span>
        </div>
      </div>
      <div class="right-controls">
        <button @click="emit('toggleFullscreen')" class="icon-button">
          <svg viewBox="0 0 24 24">
            <path d="M7 14H5v5h5v-2H7v-3zm-2-4h2V7h3V5H5v5zm12 7h-3v2h5v-5h-2v3zM14 5v2h3v3h2V5h-5z" />
          </svg>
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.controls-container {
  position: absolute;
  bottom: 5px;
  left: 0;
  right: 0;
  display: flex;
  flex-direction: column;
  color: white;
}

.progress-bar {
  width: 100%;
  cursor: pointer;
  margin: 0;
  height: 3px;
  -webkit-appearance: none;
  background: rgba(255, 255, 255, 0.3);
  transition: height 0.2s ease-in-out;
}

/* Fare kontrol alanının üzerindeyken ilerleme çubuğunu kalınlaştır */
.controls-container:hover .progress-bar {
  height: 8px;
}

.progress-bar::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 15px;
  height: 15px;
  border-radius: 50%;
  background: #fff;
  cursor: pointer;
}

.bottom-controls {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 5px 10px;
  background-color: rgba(0, 0, 0, 0.5);
}

.left-controls,
.right-controls {
  display: flex;
  align-items: center;
  gap: 15px;
}

.time-display {
  font-family: monospace;
  font-size: 14px;
}
</style>
