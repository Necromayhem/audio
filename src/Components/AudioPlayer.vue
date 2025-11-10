<script setup>
import { computed, nextTick, onMounted, ref, watch } from "vue";

const audioElement = ref(null);
const volume = ref(0.5);
const currentTrackIndex = ref(0);
const currentTime = ref(0);
const duration = ref(0);
const isLoading = ref(false);
const isPlaying = ref(false);

// картинки для обложек
const defaultImages = [
  "https://images.unsplash.com/photo-1493225457124-a3eb161ffa5f?w=300&h=300&fit=crop",
  "https://images.unsplash.com/photo-1459749411175-04bf5292ceea?w=300&h=300&fit=crop",
  "https://images.unsplash.com/photo-1514525253161-7a46d19cd819?w=300&h=300&fit=crop",
  "https://images.unsplash.com/photo-1508700115892-45ecd05ae2ad?w=300&h=300&fit=crop",
  "https://images.unsplash.com/photo-1483412033650-1015ddeb83d1?w=300&h=300&fit=crop"
];

const tracks = ref([
  {
    id: 1,
    title: "SoundHelix - Song 1",
    url: "https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3",
    img: defaultImages[0],
  },
  {
    id: 2,
    title: "SoundHelix - Song 2",
    url: "https://www.soundhelix.com/examples/mp3/SoundHelix-Song-2.mp3",
    img: defaultImages[1],
  },
  {
    id: 3,
    title: "SoundHelix - Song 3",
    url: "https://www.soundhelix.com/examples/mp3/SoundHelix-Song-3.mp3",
    img: defaultImages[2],
  },
  {
    id: 4,
    title: "SoundHelix - Song 4",
    url: "https://www.soundhelix.com/examples/mp3/SoundHelix-Song-4.mp3",
    img: defaultImages[3],
  },
  {
    id: 5,
    title: "SoundHelix - Song 5",
    url: "https://www.soundhelix.com/examples/mp3/SoundHelix-Song-5.mp3",
    img: defaultImages[4],
  }
]);

// функция воспроизведения
const play = async () => {
  if (!audioElement.value) return;
  
  try {
    // Если трек уже загружен, просто воспроизводим
    if (audioElement.value.src && audioElement.value.readyState >= 2) {
      audioElement.value.volume = volume.value;
      await audioElement.value.play();
      isPlaying.value = true;
    } else {
      // Если трек не загружен, загружаем и воспроизводим
      await playSelectedTrack();
    }
  } catch (error) {
    console.error("Ошибка воспроизведения: ", error);
    isPlaying.value = false;
  }
};

const pause = () => {
  if (audioElement.value) {
    audioElement.value.pause();
    isPlaying.value = false;
  }
};

const stop = () => {
  if (audioElement.value) {
    audioElement.value.pause();
    audioElement.value.currentTime = 0;
    isPlaying.value = false;
  }
};

const selectedTrack = async (index) => {
  if (index === currentTrackIndex.value && isPlaying.value) {
    pause();
    return;
  }
  
  currentTrackIndex.value = index;
  await playSelectedTrack();
};

const nextTrack = async () => {
  currentTrackIndex.value = (currentTrackIndex.value + 1) % tracks.value.length;
  await playSelectedTrack();
};

const prevTrack = async () => {
  currentTrackIndex.value =
    (currentTrackIndex.value - 1 + tracks.value.length) % tracks.value.length;
  await playSelectedTrack();
};

// Исправленная функция загрузки и воспроизведения трека
const playSelectedTrack = async () => {
  if (!audioElement.value) return;
  
  try {
    // Пауза перед загрузкой нового трека
    audioElement.value.pause();
    
    // Ждем следующего тика для обновления DOM
    await nextTick();
    
    // Устанавливаем громкость
    audioElement.value.volume = volume.value;
    
    // Загружаем новый источник
    audioElement.value.load();
    
    // Ждем пока трек будет готов к воспроизведению
    await new Promise((resolve, reject) => {
      const onCanPlay = () => {
        audioElement.value.removeEventListener('canplay', onCanPlay);
        resolve();
      };
      
      const onError = (error) => {
        audioElement.value.removeEventListener('error', onError);
        reject(error);
      };
      
      audioElement.value.addEventListener('canplay', onCanPlay);
      audioElement.value.addEventListener('error', onError);
      
      // Таймаут на случай если трек не загрузится
      setTimeout(() => {
        audioElement.value.removeEventListener('canplay', onCanPlay);
        audioElement.value.removeEventListener('error', onError);
        reject(new Error('Timeout loading track'));
      }, 10000);
    });
    
    // Воспроизводим трек
    await audioElement.value.play();
    isPlaying.value = true;
    
  } catch (error) {
    console.error("Ошибка загрузки/воспроизведения: ", error);
    isPlaying.value = false;
    
    // Если ошибка связана с автоплейем, просто загружаем трек
    if (error.name === 'NotAllowedError') {
      console.log('Автовоспроизведение заблокировано. Нажмите PLAY для начала воспроизведения.');
    }
  }
};

const secondsToTime = (seconds) => {
  if (!seconds || isNaN(seconds)) return "0:00";
  const mins = Math.floor(seconds / 60);
  const secs = Math.floor(seconds % 60);
  return `${mins}:${secs.toString().padStart(2, "0")}`;
};

const updateCurrentTime = () => {
  if (audioElement.value) {
    currentTime.value = audioElement.value.currentTime;
  }
};

const updateDuration = () => {
  if (audioElement.value && !isNaN(audioElement.value.duration)) {
    duration.value = audioElement.value.duration;
  }
};

const seekAudio = () => {
  if (audioElement.value) {
    audioElement.value.currentTime = currentTime.value;
  }
};

const formattedCurrentTime = computed(() => {
  return secondsToTime(currentTime.value);
});

const formattedDuration = computed(() => {
  return secondsToTime(duration.value);
});

// Обработчик окончания трека
const onTrackEnded = () => {
  isPlaying.value = false;
  nextTrack();
};

// Обработчик начала загрузки трека
const onTrackLoadStart = () => {
  isLoading.value = true;
};

// Обработчик окончания загрузки трека
const onTrackLoaded = () => {
  isLoading.value = false;
};

watch(volume, (newVolume) => {
  if (audioElement.value) {
    audioElement.value.volume = newVolume;
  }
});

watch(currentTime, (newTime) => {
  if (audioElement.value) {
    if (Math.abs(audioElement.value.currentTime - newTime) > 0.1) {
      audioElement.value.currentTime = newTime;
    }
  }
});

onMounted(() => {
  // Предзагрузка первого трека
  if (audioElement.value) {
    audioElement.value.load();
  }
});
</script>

<template>
  <div class="main">
    <audio
      ref="audioElement"
      :src="tracks[currentTrackIndex]?.url"
      @timeupdate="updateCurrentTime"
      @loadedmetadata="updateDuration"
      @ended="onTrackEnded"
      @loadstart="onTrackLoadStart"
      @loadeddata="onTrackLoaded"
      preload="metadata"
    ></audio>
    
    <div class="player-container">
      <!-- Индикатор загрузки -->
      <div v-if="isLoading" class="loading">
        <div class="spinner"></div>
        Загрузка трека...
      </div>

      <div class="content">
        <div class="cover">
          <img 
            :src="tracks[currentTrackIndex]?.img" 
            :alt="tracks[currentTrackIndex]?.title"
            @error="(e) => { e.target.src = defaultImages[0] }"
          >
        </div>
        
        <div class="track-info">
          <h3>{{ tracks[currentTrackIndex]?.title }}</h3>
          <div class="track-status">
            {{ isPlaying ? 'Воспроизведение' : 'Пауза' }} • 
            {{ currentTrackIndex + 1 }} из {{ tracks.length }}
          </div>
        </div>
        
        <div class="main-controls">
          <button @click="prevTrack" title="Предыдущий трек">⏮</button>
          <button 
            @click="isPlaying ? pause() : play()" 
            :class="{ playing: isPlaying }"
            class="play-pause-btn"
            :title="isPlaying ? 'Пауза' : 'Воспроизведение'"
          >
            {{ isPlaying ? '⏸️' : '▶️' }}
          </button>
          <button @click="stop" title="Стоп">⏹️</button>
          <button @click="nextTrack" title="Следующий трек">⏭</button>
        </div>

        <div class="volume-control">
          <span>🔊</span>
          <input 
            type="range" 
            min="0" 
            max="1" 
            step="0.01" 
            v-model="volume" 
            title="Громкость"
          />
          <span class="volume-percentage">{{ Math.round(volume * 100) }}%</span>
        </div>
        
        <div class="duration">
          <span class="time">{{ formattedCurrentTime }}</span>
          <input 
            type="range" 
            min="0" 
            :max="duration || 0" 
            step="1" 
            v-model="currentTime" 
            @input="seekAudio"
            :disabled="!duration"
          />
          <span class="time">{{ formattedDuration }}</span>
        </div>
        
        <div class="track-list">
          <div
            v-for="(track, index) in tracks"
            :key="track.id"
            @click="selectedTrack(index)"
            :class="{ 
              active: currentTrackIndex === index,
              playing: currentTrackIndex === index && isPlaying
            }"
            class="track-item"
          >      
            <span class="track-number">{{ index + 1 }}.</span>
            <span class="track-title">{{ track.title }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.main {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: 100vh;
  margin-top: 50px;
  padding: 20px;
}

.player-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  max-width: 600px;
  width: 100%;
  gap: 20px;
  background: linear-gradient(135deg, #262626 0%, #1a1a1a 100%);
  padding: 30px;
  border-radius: 20px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
  border: 1px solid #333;
}

.loading {
  display: flex;
  align-items: center;
  gap: 10px;
  color: #1DB954;
  font-size: 16px;
  font-weight: bold;
  padding: 20px;
}

.spinner {
  width: 20px;
  height: 20px;
  border: 2px solid #333;
  border-top: 2px solid #1DB954;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.content {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
  width: 100%;
}

.cover {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
  
  img {
    width: 250px;
    height: 250px;
    object-fit: cover;
    border-radius: 15px;
    border: 3px solid #1DB954;
    box-shadow: 0 8px 25px rgba(29, 185, 84, 0.3);
  }
}

.playing-indicator {
  color: #1DB954;
  font-weight: bold;
  font-size: 14px;
  animation: pulse 2s infinite;
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.7; }
}

.track-info {
  text-align: center;
  
  h3 {
    color: white;
    margin: 0 0 8px 0;
    font-size: 20px;
    font-weight: 600;
  }
}

.track-status {
  color: #888;
  font-size: 14px;
}

.main-controls {
  display: flex;
  gap: 15px;
  align-items: center;
}

.main-controls button {
  background: #333;
  color: white;
  border: none;
  padding: 12px;
  border-radius: 50%;
  font-size: 16px;
  cursor: pointer;
  transition: all 0.3s ease;
  width: 50px;
  height: 50px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.main-controls button:hover {
  background: #444;
  transform: scale(1.1);
}

.play-pause-btn {
  font-size: 18px;
  :hover {
    background: #1DB954 
  }
}

.play-pause-btn:hover {
  background: #1ed760 !important;
  transform: scale(1.15);
}

.volume-control {
  display: flex;
  align-items: center;
  gap: 12px;
  color: white;
  width: 100%;
  max-width: 200px;
}

.volume-control input {
  flex: 1;
}

.volume-percentage {
  font-size: 12px;
  color: #888;
  min-width: 40px;
}

.duration {
  display: flex;
  align-items: center;
  gap: 15px;
  width: 100%;
  
  .time {
    color: white;
    font-size: 14px;
    min-width: 45px;
    font-variant-numeric: tabular-nums;
  }
}

.duration input {
  flex: 1;
  height: 6px;
}

.track-list {
  width: 100%;
  max-height: 300px;
  overflow-y: auto;
  border-radius: 10px;
  background: #1a1a1a;
  border: 1px solid #333;
}

.track-item {
  display: flex;
  align-items: center;
  gap: 12px;
  color: white;
  padding: 12px 16px;
  cursor: pointer;
  border-bottom: 1px solid #2a2a2a;
  transition: all 0.3s ease;
}

.track-item:hover {
  background: #2a2a2a;
}

.track-item.active {
  background: #333;
}

.track-item.playing {
  background: linear-gradient(135deg, #1DB95420 0%, #1DB95410 100%);
  border-left: 3px solid #1DB954;
}

.track-number {
  color: #888;
  font-size: 14px;
  min-width: 25px;
}

.track-title {
  flex: 1;
  font-size: 14px;
}

.now-playing {
  color: #1DB954;
  font-size: 12px;
}

.track-item:last-child {
  border-bottom: none;
}

/* Стили для ползунков */
input[type="range"] {
  -webkit-appearance: none;
  background: #555;
  border-radius: 5px;
  cursor: pointer;
  height: 6px;
}

input[type="range"]:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  background: #1DB954;
  width: 16px;
  height: 16px;
  border-radius: 50%;
  cursor: pointer;
  transition: all 0.2s ease;
}

input[type="range"]::-webkit-slider-thumb:hover {
  transform: scale(1.2);
}

input[type="range"]::-moz-range-thumb {
  background: #1DB954;
  width: 16px;
  height: 16px;
  border-radius: 50%;
  border: none;
  cursor: pointer;
}

/* Скроллбар */
.track-list::-webkit-scrollbar {
  width: 8px;
}

.track-list::-webkit-scrollbar-track {
  background: #1a1a1a;
}

.track-list::-webkit-scrollbar-thumb {
  background: #1DB954;
  border-radius: 4px;
}

.track-list::-webkit-scrollbar-thumb:hover {
  background: #1ed760;
}
</style>