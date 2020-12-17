<template>
  <Chart />
  <div class="keyboard">
    <div
      class="key"
      :class="{ black: pitch.class[pitch.class.length - 1] === '#', selected: pitch.selected }"
      v-for="pitch in pitches"
      :key="pitch.frequency"
      @click="pitch.selected = !pitch.selected"
    />
  </div>
</template>

<script>
import Chart from './Chart.vue'

export default {
  components: {
    Chart
  },
  data() {
    return {
      pitches: [],
    }
  },
  created() {
    let classes = ['A', 'A#', 'B', 'C', 'C#', 'D', 'D#', 'E', 'F', 'F#', 'G', 'G#']
    let baseFreq = 27.5
    let numPitches = 88
    for (let i = 0; i < numPitches; ++i) {
      this.pitches.push({
        class: classes[i%12],
        frequency: baseFreq,
        selected: false,
      })
      baseFreq *= 2 ** (1/12)
    }
  }
}
</script>

<style scoped>
.keyboard {
  display: flex;
  flex-direction: row;
  overflow: auto;
}
.key {
  width: 24px;
  min-width: 24px;
  height: 144px;
  min-height: 144px;
  background-color: white;
  border: 2px solid #1f1f1f;
}
.black {
  background-color: #1f1f1f;
}
.selected {
  border: 2px solid lightskyblue;
}
</style>