<script setup>
import { ref, computed, toRefs, onMounted } from "vue";
import { highscoresConfig } from "./highscores_config";

const props = defineProps({
  gameId: {
    type: String,
    required: false,
    default: "",
  },
});
const { gameId } = toRefs(props);

const apiBase = import.meta.env.PUBLIC_API_BASE || "https://api.jakspeedruns.workers.dev";

// state
const loading = ref(false);
const highscoresToRender = ref([]);
const pausedFilter = ref("all"); // "all" | "paused" | "unpaused"

const filteredHighscores = computed(() => {
  return highscoresToRender.value.map((highscore) => {
    let data = highscore.data;
    if (pausedFilter.value === "paused") {
      data = data.filter((s) => s.paused === true);
    } else if (pausedFilter.value === "unpaused") {
      data = data.filter((s) => s.paused === false);
    }
    return { ...highscore, data };
  });
});

onMounted(async () => {
  loading.value = true;
  let highscores = highscoresConfig[gameId.value];
  for (let highscore of highscores) {
    highscore.data = await fetch(`${apiBase}/v1/highscores/${highscore.id}`).then((res) => res.json());
  }
  highscoresToRender.value = highscores;
  loading.value = false;
});
</script>

<template>
  <div v-if="loading">
    Loading...
  </div>
  <template v-else>
    <div class="paused-filter">
      <label>Show:</label>
      <label><input type="radio" value="all" v-model="pausedFilter" /> All</label>
      <label><input type="radio" value="unpaused" v-model="pausedFilter" /> Unpaused only</label>
      <label><input type="radio" value="paused" v-model="pausedFilter" /> Paused only</label>
    </div>
    <div v-for="highscore in filteredHighscores" :key="highscore.id">
      <h2>{{ highscore.name }}</h2>
      <table class="styled-table">
        <thead>
          <tr>
            <th>Rank</th>
            <th>Player</th>
            <th>Score</th>
            <th>Paused</th>
            <th>Video</th>
            <th>Date</th>
          </tr>
        </thead>
        <tbody>
          <tr v-if="highscore.data.length === 0">
            <td colspan="6">No Scores Found</td>
          </tr>
          <tr v-else v-for="(score, index) in highscore.data" :key="`${highscore.id}-${index}`" :class="score.points === null ? 'no-background' : null">
            <td>{{ index + 1 }}</td>
            <td>{{ score.playerName }}</td>
            <td>{{ score.score }}</td>
            <td>{{ score.paused ? "Yes" : "No" }}</td>
            <td v-if="score.videoLink !== null"><a :href="score.videoLink" target="_blank" rel="noopener">{{ score.videoLink }}</a></td>
            <td v-else>N/A</td>
            <td v-if="score.timestamp !== null">{{ score.timestamp }}</td>
            <td v-else>N/A</td>
          </tr>
        </tbody>
      </table>
    </div>
  </template>
</template>

<style scoped>
.paused-filter {
  display: flex;
  gap: 1rem;
  align-items: center;
  margin-bottom: 1rem;
  flex-wrap: wrap;
}
.paused-filter label {
  cursor: pointer;
}
</style>
