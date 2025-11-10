<script setup>
import { computed, nextTick, onMounted, ref, watch } from "vue";

const audioElement = ref(null);
const volume = ref(0.5);
const currentTrackIndex = ref(0);
const currentTime = ref(0);
const duration = ref(0);
const isLoading = ref(false);
const error = ref(null);

const tracks = ref([
  {
    url: "/audio/None - A World, Dead and Gray.mp3", 
    title: "None - A World, Dead and Gray",
    img: "/img/none.jfif",
  },
  {
    url: "/audio/Alcest - Sur l'océan couleur de fer.mp3",
    title: "Alcest - Sur l'océan couleur de fer",
    img: "/img/Alcest_—_Écailles_de_Lune.jpg",
  },
  {
    url: "/audio/December's Fire - Nostalgia.mp3",
    title: "December's Fire - Nostalgia",
    img: "/img/R-2647127-1709751321-8277.jpg",
  },
]);

const play = async () => {
  try {
    if (!audioElement.value) return;
    
    audioElement.value.volume = volume.value;
    await audioElement.value.play();
    error.value = null;
  } catch (err) {
    console.error("Ошибка воспроизведения: ", err);
    error.value = "Ошибка воспроизведения: " + err.message;
  }
};

const pause = () => {
  if (audioElement.value) {
    audioElement.value.pause();
  }
};

const stop = () => {
  if (audioElement.value) {
    audioElement.value.pause();
    audioElement.value.currentTime = 0;
    currentTime.value = 0;
  }
};

const selectedTrack = async (index) => {
  if (index === currentTrackIndex.value && audioElement.value) {
    // Если выбрали текущий трек, перезапускаем
    audioElement.value.currentTime = 0;
    await play();
  } else {
    currentTrackIndex.value = index;
    await playSelectedTrack();
  }
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

const playSelectedTrack = async () => {
  try {
    isLoading.value = true;
    error.value = null;
    
    await nextTick();
    
    if (!audioElement.value) {
      throw new Error("Audio element not found");
    }

    // Сбрасываем текущее время
    currentTime.value = 0;
    
    // Даем время на обновление src
    await new Promise(resolve => setTimeout(resolve, 100));
    
    // Загружаем новый трек
    audioElement.value.load();
    audioElement.value.volume = volume.value;
    
    // Ждем пока аудио будет готово
    await new Promise((resolve, reject) => {
      if (!audioElement.value) return reject();
      
      const onCanPlay = () => {
        audioElement.value.removeEventListener('canplay', onCanPlay);
        resolve();
      };
      
      const onError = () => {
        audioElement.value.removeEventListener('error', onError);
        reject(new Error("Failed to load audio"));
      };
      
      audioElement.value.addEventListener('canplay', onCanPlay);
      audioElement.value.addEventListener('error', onError);
      
      // Таймаут на случай если загрузка зависнет
      setTimeout(() => {
        audioElement.value.removeEventListener('canplay', onCanPlay);
        audioElement.value.removeEventListener('error', onError);
        reject(new Error("Audio loading timeout"));
      }, 10000);
    });
    
    await audioElement.value.play();
    
  } catch (err) {
    console.error("Ошибка воспроизведения: ", err);
    error.value = "Ошибка загрузки трека: " + err.message;
  } finally {
    isLoading.value = false;
  }
};

const secondsToTime = (seconds) => {
  if (!seconds || isNaN(seconds)) return "0:00";
  const mins = Math.floor(seconds / 60);
  const secs = Math.floor(seconds % 60);
  return `${mins}:${secs.toString().padStart(2, "0")}`;
};

const updateCurrentTime = () => {
  if (audioElement.value && !isNaN(audioElement.value.currentTime)) {
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
}

const formattedCurrentTime = computed(() => {
  return secondsToTime(currentTime.value);
});

const formattedDuration = computed(() => {
  return secondsToTime(duration.value);
});

watch(volume, (newVolume) => {
  if (audioElement.value) {
    audioElement.value.volume = newVolume;
  }
});

// может вызывать рекурсию
// watch(currentTime, (newTime) => {
//   if (audioElement.value) {
//     if (Math.abs(audioElement.value.currentTime - newTime) > 0.1) {
//       audioElement.value.currentTime = newTime;
//     }
//   }
// });

onMounted(() => {
  if (audioElement.value) {
    audioElement.value.load();
  }
});
</script>

<template>
  <div class="main">
    <audio
      ref="audioElement"
      :src="tracks[currentTrackIndex].url"
      @timeupdate="updateCurrentTime"
      @canplaythrough="updateDuration"
      @ended="nextTrack"
    ></audio>
    <div class="player-container">
      <div class="cover">
        <img :src="tracks[currentTrackIndex].img" alt="">
      </div>
      <div class="main-controls">
        <button @click="play">PLAY</button>
        <button @click="pause">PAUSE</button>
        <button @click="stop">STOP</button>
        <button @click="prevTrack">prev</button>
        <button @click="nextTrack">next</button>
        <input type="range" min="0" max="1" step="0.01" v-model="volume" />
      </div>
      
      <div class="duration">
        <span>{{ formattedCurrentTime }}</span>
        <input 
          type="range" 
          min="0" 
          :max="duration" 
          step="1" 
          v-model="currentTime" 
          @input="seekAudio"
        />
        <span>{{ formattedDuration }}</span>
      </div>
       <div class="track-list">
      <li
        v-for="(track, index) in tracks"
        :key="index"
        @click="selectedTrack(index)"
        :class="{ active: currentTrackIndex === index }"
      >      
        {{ track.title }}
      </li>
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
  margin-top: 200px;
}

.player-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  max-width: 500px;
  gap: 20px;
  background-color: #262626;
  padding: 20px;
}

.main-controls {
  display: flex;
  gap: 10px;
  align-items: center;
}

.duration {
  display: flex;
  align-items: center;
  gap: 10px;
  span {
    color: white;
  }
}

.duration input {
  width: 400px;
}

.track-list {
  cursor: pointer;
  :hover {
    color:red;
  }

  li {
    color: white;
    list-style-type: none;
  }
}

.main-controls button, input {
  cursor: pointer;
}

.duration input {
  cursor: pointer;
}



.cover {
  display: flex;
  justify-content: center;
  width: 500px;
  img {
  width: 200px;
  height: 200px;
  object-fit: contain;
}
}

</style>
