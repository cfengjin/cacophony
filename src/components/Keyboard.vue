<template>
  <Chart :pitches="selectedPitches" :showFundamentals="showFundamentals" :showInterference="showInterference"/>

  <div class="keyboard">
    <div
      class="key"
      :class="{ black: isBlack(pitch) }"
      :style="{ backgroundColor: keyColor(pitch)}"
      v-for="pitch in pitches"
      :key="pitch.frequency"
      @click="pitch.selected = !pitch.selected; stopSequence()"
    />
  </div>

  <div class="controls">
    <div>
      Display:
      <label for="fundamentals">
        <input type="checkbox" id="fundamentals" v-model="showFundamentals">
        Fundamentals
      </label>
      <label for="interference">
        <input type="checkbox" id="interference" v-model="showInterference">
        Interference
      </label>
    </div>

    <div>
      Waveform:
      <label for="sine">
        <input type="radio" id="sine" value="sine" v-model="waveform">
        Sine
      </label>
      <label for="square">
        <input type="radio" id="square" value="square" v-model="waveform">
        Square
      </label>
      <label for="triangle">
        <input type="radio" id="triangle" value="triangle" v-model="waveform">
        Triangle
      </label>
      <label for="sawtooth">
        <input type="radio" id="sawtooth" value="sawtooth" v-model="waveform">
        Sawtooth
      </label>
    </div>

    <div>
      <label for="volume">Volume:</label>
      <input type="range" id="volume" min="0" max="1" step="0.01" v-model="volume">
    </div>

    <div>
      <button @click.prevent="clear(true)">Clear</button>
      <button @mousedown="play(true)" @touchstart.prevent="play(true)">Play</button>
    </div>
  </div>

  <div class="sequencer">
    <button @click.prevent="clearChords">Clear Chords</button>
    <div
      class="chord"
      @click.prevent="selectChord(index)"
      v-for="(chord, index) in chords"
      :key="chord"
    >
      <div v-for="pitch in chord.slice().reverse()" :key="pitch.frequency">
        {{ pitch.class }}
      </div>
      <span @click.stop.prevent="removeChord(index)">&#10006;</span>
    </div>
    <button @click.prevent="addChord">Add Chord</button>
    <button @click.prevent="startSequence">Start</button>
    <button @click.prevent="stopSequence">Stop</button>
    <div>
      <label for="volume">Tempo:</label>
      <input type="number" id="volume" min="40" max="208" step="1" v-model="tempo">
    </div>
  </div>
</template>

<script>
import Chart from './Chart.vue'
import * as d3 from "d3"

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
      playing: false,
      showFundamentals: true,
      showInterference: false,
      chords: [],
      tempo: 76,
      sequencePosition: 0,
      intervalID: 0,
    }
  },
  computed: {
    selectedPitches() {
      return this.pitches.filter(pitch => pitch.selected)
    },
    color() {
      return d3.scaleOrdinal()
        .domain(['A', 'A#', 'B', 'C', 'C#', 'D', 'D#', 'E', 'F', 'F#', 'G', 'G#'])
        .range(['#4e79a7', '#f28e2c', '#e15759', '#76b7b2', '#59a14f', '#edc949', '#af7aa1', '#ff9da7', '#9c755f', '#bab0ab', '#888888', 'whitesmoke'])
    }
  },
  watch: {
    volume() {
      this.masterGainNode.gain.setValueAtTime(this.volume, this.audioContext.currentTime)
    },
    tempo() {
      this.stopSequence()
      if (this.intervalID !== 0) {
        this.sequence()
      }
    }
  },
  methods: {
    play(click) {
      if (click) {
        this.stopSequence()
      }
      console.log("hello")
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
    clear(click) {
      if (click) {
        this.stopSequence()
      }
      this.selectedPitches.forEach(pitch => pitch.selected = false)
    },
    isBlack(pitch) {
      return pitch.class[pitch.class.length - 1] === '#'
    },
    keyColor(pitch) {
      return pitch.selected ? this.color(pitch.class) : this.isBlack(pitch) ? '#1f1f1f' : 'white'
    },
    addChord() {
      if (this.selectedPitches.length > 0) {
        this.chords.push(this.selectedPitches);
      }
    },
    selectChord(index) {
      this.clear(false);
      this.chords[index].forEach(pitch => pitch.selected = true);
    },
    playChord() {
      this.stop();
      this.clear(false);
      this.chords[this.sequencePosition++].forEach(pitch => pitch.selected = true);
      this.play(false);
      if (this.sequencePosition >= this.chords.length) {
        this.sequencePosition = 0
      }
    },
    startSequence() {
      this.stopSequence()
      this.sequencePosition = 0
      this.sequence()
    },
    sequence() {
      if (this.chords.length > 0) {
        this.playChord()
        let self = this
        this.intervalID = setInterval(function() {
          self.playChord()
        }, 60000 / this.tempo)
      }
    },
    stopSequence() {
      if (this.intervalID !== 0) {
        clearInterval(this.intervalID);
        this.intervalID = 0;
        this.stop()
      }
    },
    removeChord(index) {
      if (this.chords.length === 1) {
        this.stopSequence()
      }
      this.chords.splice(index, 1);
      if (this.sequencePosition >= this.chords.length) {
        this.sequencePosition = 0
      }
    },
    clearChords() {
      this.stopSequence()
      this.chords = [];
    },
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
        class: classes[i % 12],
        frequency: baseFreq,
        selected: false,
        oscillator: {},
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
.controls div {
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

.sequencer {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-wrap: wrap;
  margin-bottom: 24px;
}

.sequencer button {
  width: 5rem;
  height: 3rem;
}

.chord {
  margin: 0 4px;
  align-self: flex-end;
}

</style>