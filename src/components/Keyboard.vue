<template>
  <Chart :pitches="selectedPitches" />

  <div class="keyboard">
    <div
      class="key"
      :class="{ black: pitch.class[pitch.class.length - 1] === '#', selected: pitch.selected }"
      v-for="pitch in pitches"
      :key="pitch.frequency"
      @click="pitch.selected = !pitch.selected"
    />
  </div>

  <div class="controls">
    <div class="left">
      <label for="waveform">Waveform:</label>
      <label for="sine">
        <input type="radio" id="sine" name="waveform" value="sine" v-model="waveform">
        Sine
      </label>
      <label for="square">
        <input type="radio" id="square" name="waveform" value="square" v-model="waveform">
        Square
      </label>
      <label for="triangle">
        <input type="radio" id="triangle" name="waveform" value="triangle" v-model="waveform">
        Triangle
      </label>
      <label for="sawtooth">
        <input type="radio" id="sawtooth" name="waveform" value="sawtooth" v-model="waveform">
        Sawtooth
      </label>
    </div>

    <div class="center">
      <label for="volume">Volume:</label>
      <input type="range" id="volume" name="volume" min="0" max="1" step="0.01" v-model="volume">
    </div>

    <div class="right">
      <button @click="clear">Clear</button>
      <button @mousedown="play" @touchstart="play">Play</button>
    </div>
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
      audioContext: new (window.AudioContext || window.webkitAudioContext)(),
      masterGainNode: {},
      volume: 0.1,
      pitches: [],
      waveform: 'sine',
      playing: false
    }
  },
  computed: {
    selectedPitches() {
      return this.pitches.filter(pitch => pitch.selected)
    }
  },
  watch: {
    volume() {
      this.masterGainNode.gain.value = this.volume
    }
  },
  methods: {
    play() {
      this.playing = true
      this.selectedPitches.forEach(pitch => {
        pitch.oscillator = this.audioContext.createOscillator()
        pitch.oscillator.frequency.value = pitch.frequency
        pitch.oscillator.type = this.waveform
        pitch.oscillator.connect(this.masterGainNode)
        pitch.oscillator.start()
      })
    },
    stop() {
      if (this.playing) {
        this.selectedPitches.forEach(pitch => pitch.oscillator.stop())
        this.playing = false
      }
    },
    clear() {
      this.selectedPitches.forEach(pitch => pitch.selected = false)
    }
  },
  created() {
    this.masterGainNode = this.audioContext.createGain()
    this.masterGainNode.connect(this.audioContext.destination)
    this.masterGainNode.gain.value = this.volume
    let classes = ['A', 'A#', 'B', 'C', 'C#', 'D', 'D#', 'E', 'F', 'F#', 'G', 'G#']
    let baseFreq = 27.5
    let numPitches = 88
    for (let i = 0; i < numPitches; ++i) {
      this.pitches.push({
        class: classes[i%12],
        frequency: baseFreq,
        selected: false,
        oscillator: {} 
      })
      baseFreq *= 2 ** (1/12)
    }
    window.addEventListener('mouseup', this.stop)
    window.addEventListener('touchend', this.stop)
  },
  mounted() {
    let keyboard = document.querySelector(".keyboard")
    let inner = keyboard.scrollWidth
    let outer = keyboard.offsetWidth
    keyboard.scrollLeft = (inner - outer) / 2
  }
}
</script>

<style scoped>
.keyboard {
  display: flex;
  flex-direction: row;
  overflow: auto;
  height: 182px;
  min-height: 182px;
  margin-bottom: 24px;
}
.key {
  margin: 0 2px;
  width: 30px;
  min-width: 30px;
  height: 180px;
  min-height: 180px;
  background-color: white;
  border: 1px solid #1f1f1f;
}
.black {
  width: 18px;
  min-width: 18px;
  height: 108px;
  min-height: 108px;
  background-color: #1f1f1f;
}
.selected {
  background-color: deepskyblue;
  border-color: deepskyblue;
}
.controls {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-wrap: wrap;
  margin-bottom: 24px;
}
.controls .left, .center, .right {
  display: flex;
  justify-content: space-evenly;
  align-items: center;
  flex-wrap: wrap;
  margin: 0 24px;
}
.controls button {
  width: 5rem;
  height: 3rem;
}
</style>