<script setup>
import { computed, nextTick, onMounted, ref, watch } from "vue";
import track2 from "./audio/Alcest - Sur l'océan couleur de fer.mp3";
import track3 from "./audio/December's Fire - Nostalgia.mp3";
import track1 from "./audio/None - A World, Dead and Gray.mp3";

const audioElement = ref(null);
const volume = ref(0.5);
const currentTrackIndex = ref(0);
const currentTime = ref(0);
const duration = ref(0);

const tracks = ref([
  {
    url: track1,
    title: "None - A World, Dead and Gray",
    img: "",
  },
  {
    url: track2,
    title: "Alcest - Sur l'océan couleur de fer",
    img: "",
  },
  {
    url: track3,
    title: "December's Fire - Nostalgia",
    img: "",
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

onMounted(() => {
  audioElement.value.load();
});
</script>

<template>
  <div class="main"></div>
  <audio
    ref="audioElement"
    :src="tracks[currentTrackIndex].url"
    @timeupdate="updateCurrentTime"
    @canplaythrough="updateDuration"
  ></audio>
  <div class="controls">
    <button @click="play">PLAY</button>
    <button @click="pause">PAUSE</button>
    <button @click="stop">STOP</button>
    <button @click="prevTrack">prev</button>
    <button @click="nextTrack">next</button>
    <input type="range" min="0" max="1" step="0.01" v-model="volume" />
    <span>{{ formattedCurrentTime }}</span>
    <input type="range" min="0" max="" />
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
</template>

<style scoped>
.track-list {
  cursor: pointer;
}

.controls button {
  cursor: pointer;
}
</style>
