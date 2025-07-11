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

const toggleVoiceRecognition = () => {
  if (!recognition) {
    console.error('Speech Recognition is not available.');
    return;
  }
  if (isListening.value) {
    isListening.value = false; // Signal intentional stop
    recognition.stop();
  } else {
    try {
      recognition.start();
    } catch (error) {
      console.error('Could not start voice recognition:', error);
    }
  }
};

onMounted(() => {
  if (videoElement.value) {
    const setInitialValues = () => {
      duration.value = videoElement.value.duration;
      volume.value = videoElement.value.volume;
      isMuted.value = videoElement.value.muted;
    };

    videoElement.value.addEventListener('loadedmetadata', setInitialValues);
    videoElement.value.addEventListener('durationchange', setInitialValues);
    videoElement.value.addEventListener('timeupdate', () => { currentTime.value = videoElement.value.currentTime; });
    videoElement.value.addEventListener('play', () => { isPlaying.value = true; });
    videoElement.value.addEventListener('pause', () => { isPlaying.value = false; });
    videoElement.value.addEventListener('ended', () => { isPlaying.value = false; });
    videoElement.value.addEventListener('volumechange', () => {
      volume.value = videoElement.value.volume;
      isMuted.value = videoElement.value.muted;
    });

    if (videoElement.value.readyState >= 1) {
      setInitialValues();
    }
  }

  const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
  if (SpeechRecognition) {
    recognition = new SpeechRecognition();
    recognition.continuous = true;
    recognition.lang = 'tr-TR';
    recognition.interimResults = false;

    recognition.onstart = () => {
      isListening.value = true;
      console.log('Voice recognition started (onstart fired).');
    };

    recognition.onend = () => {
      console.log(`Voice recognition ended (onend fired). isListening state: ${isListening.value}`);
      if (isListening.value) {
        console.log('Voice recognition service ended, restarting...');
        try {
          recognition.start();
        } catch (error) {
          console.error('Failed to restart recognition:', error);
          isListening.value = false;
        }
      } else {
        console.log('Voice recognition stopped intentionally.');
      }
    };

    recognition.onerror = (event) => {
      console.error(`Speech recognition error (onerror fired). Error: ${event.error}`, event);
      isListening.value = false;
    };

    recognition.onresult = (event) => {
      console.log(`onresult fired. Results length: ${event.results.length}`);
      let final_transcript = '';
      for (let i = event.resultIndex; i < event.results.length; ++i) {
        console.log(`Result ${i}: isFinal = ${event.results[i].isFinal}, Transcript = '${event.results[i][0].transcript}'`);
        if (event.results[i].isFinal) {
          final_transcript += event.results[i][0].transcript;
        }
      }
      const command = final_transcript.trim().toLowerCase();
      if (command) {
        console.log('Final command received:', command);
        handleVoiceCommand(command);
      } else {
        console.log('No final command processed from this result.');
      }
    };
  } else {
    console.warn('Web Speech API is not supported in this browser.');
  }
});

onUnmounted(() => {
  if (recognition) {
    isListening.value = false;
    recognition.stop();
    recognition = null;
  }
});

const handleVoiceCommand = (command) => {
  const commandMap = {
    'oynat': () => { if (!isPlaying.value) videoElement.value.play(); },
    'duraklat': () => { if (isPlaying.value) videoElement.value.pause(); },
    'başa sar': () => { seek(0); },
    'ileri sar': () => { seek(Math.min(duration.value, currentTime.value + 5)); },
    'geri sar': () => { seek(Math.max(0, currentTime.value - 5)); },
  };

  const action = commandMap[command];
  if (action) {
    action();
  } else {
    console.log('Unknown command:', command);
  }
};

const togglePlayPause = () => {
  if (isPlaying.value) {
    videoElement.value.pause();
  } else {
    videoElement.value.play();
  }
};

const seek = (time) => {
  if (videoElement.value) {
    videoElement.value.currentTime = time;
  }
};

const updateVolume = (newVolume) => {
  if (videoElement.value) {
    videoElement.value.volume = newVolume;
  }
};

const toggleMute = () => {
  if (videoElement.value) {
    videoElement.value.muted = !videoElement.value.muted;
  }
};

const toggleFullscreen = () => {
  if (videoElement.value.requestFullscreen) {
    videoElement.value.requestFullscreen();
  } else if (videoElement.value.mozRequestFullScreen) { /* Firefox */
    videoElement.value.mozRequestFullScreen();
  } else if (videoElement.value.webkitRequestFullscreen) { /* Chrome, Safari & Opera */
    videoElement.value.webkitRequestFullscreen();
  } else if (videoElement.value.msRequestFullscreen) { /* IE/Edge */
    videoElement.value.msRequestFullscreen();
  }
};

</script>

<template>
  <div class="player-container">
    <div class="video-player">
      <video ref="videoElement" @dblclick="toggleFullscreen">
        <source :src="props.videoUrl" type="video/mp4">
      </video>

      <Controls :isPlaying="isPlaying" :currentTime="currentTime" :duration="duration" :volume="volume" :isMuted="isMuted"
        @togglePlayPause="togglePlayPause" @seek="seek" @updateVolume="updateVolume" @toggleMute="toggleMute"
        @toggleFullscreen="toggleFullscreen" />
    </div>
    <button @click="toggleVoiceRecognition" class="voice-btn">
      {{ isListening ? '🎙️ Sesli Komutu Bitir' : '🎙️ Sesli Komutu Başlat' }}
    </button>
    <div v-if="isListening" class="voice-commands">
      <h4>Kullanılabilir Sesli Komutlar</h4>
      <ul>
        <li><b>"oynat"</b> - Videoyu başlatır</li>
        <li><b>"duraklat"</b> - Videoyu duraklatır</li>
        <li><b>"başa sar"</b> - Videoyu en başa sarar</li>
        <li><b>"ileri sar"</b> - 5 saniye ileri sarar</li>
        <li><b>"geri sar"</b> - 5 saniye geri sarar</li>
      </ul>
    </div>
  </div>
</template>

<style scoped>
.player-container {
  width: 100%;
  max-width: 640px;
  margin: 0 auto;
}

.video-player {
  position: relative;
  width: 100%;
  padding-top: 56.25%; /* 16:9 Aspect Ratio */
  background-color: black;
}

.video-player video {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

.voice-btn {
  display: block;
  margin: 15px auto;
  background-color: #dc3545;
  color: white;
  border: none;
  padding: 10px 16px;
  border-radius: 5px;
  cursor: pointer;
  font-size: 16px;
  z-index: 10;
}

.voice-btn:hover {
  background-color: #c82333;
}

.voice-commands {
  margin-top: 15px;
  padding: 15px;
  background-color: #f8f9fa;
  border: 1px solid #dee2e6;
  border-radius: 5px;
  text-align: center;
}

.voice-commands h4 {
  margin-top: 0;
  margin-bottom: 10px;
  font-size: 18px;
  color: #333;
}

.voice-commands ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

.voice-commands li {
  font-size: 16px;
  color: #555;
  margin-bottom: 5px;
}
</style>
