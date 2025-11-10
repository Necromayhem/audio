<script setup>
import { computed, nextTick, onMounted, ref, watch } from "vue";

const audioElement = ref(null);
const volume = ref(0.5);
const currentTrackIndex = ref(0);
const currentTime = ref(0);
const duration = ref(0);

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
    audioElement.value.volume = volume.value;
    await audioElement.value.play();
  } catch (error) {
    console.error("Ошибка воспроизведения: ", error);
  }
};

const pause = () => {
  audioElement.value.pause();
};

const stop = () => {
  audioElement.value.pause();
  audioElement.value.currentTime = 0;
};

const selectedTrack = async (index) => {
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

const playSelectedTrack = async () => {
  try {
    await nextTick();
    if (audioElement.value) {
      await audioElement.value.load();
      audioElement.value.volume = volume.value;
      await audioElement.value.play();
    }
  } catch (error) {
    console.error("Ошибка воспроизведения: ", error);
  }
};

const secondsToTime = (seconds) => {
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
  if (audioElement.value) {
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

watch(currentTime, (newTime) => {
  if (audioElement.value) {
    if (Math.abs(audioElement.value.currentTime - newTime) > 0.1) {
      audioElement.value.currentTime = newTime;
    }
  }
});

onMounted(() => {
  audioElement.value.load();
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
