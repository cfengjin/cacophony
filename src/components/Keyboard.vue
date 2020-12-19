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

  <button @mousedown="play">Play</button>
  <button @click="clear">Clear</button>

  <br />
  <label for="volume">Volume:</label>
  <br />
  <input type="range" id="volume" name="volume" min="0" max="1" step="0.01" v-model="volume">

  <p>Select a waveform:</p>

  <input type="radio" id="sine" name="waveform" value="sine" v-model="waveform">
  <label for="sine">Sine</label>
  <br />
  <input type="radio" id="square" name="waveform" value="square" v-model="waveform">
  <label for="square">Square</label>
  <br />
  <input type="radio" id="triangle" name="waveform" value="triangle" v-model="waveform">
  <label for="triangle">Triangle</label>
  <br />
  <input type="radio" id="sawtooth" name="waveform" value="sawtooth" v-model="waveform">
  <label for="sawtooth">Sawtooth</label>

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
      this.selectedPitches.forEach(pitch => {
        pitch.oscillator = this.audioContext.createOscillator()
        pitch.oscillator.frequency.value = pitch.frequency
        pitch.oscillator.type = this.waveform
        pitch.oscillator.connect(this.masterGainNode)
        pitch.oscillator.start()
      })
      this.playing = true
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
  },
}
</script>

<style scoped>
.keyboard {
  display: flex;
  flex-direction: row;
  overflow: auto;
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
</style>