<template>
  <div class="chart"/>
</template>

<script>
import * as d3 from "d3"

export default {
  props: ['pitches'],
  data() {
    return {
      svg: {}
    }
  },
  computed: {
    data() {
      let delta = .001;
      return this.pitches.map(pitch => {
        let points = [];
        for (let i = 0; i < 1/delta; ++i) {
          let t = delta*i*this.graph.domain[1]
          points.push({
            time: t,
            displacement: this.d(pitch.frequency, t)
          })
        }
        return {
          class: pitch.class,
          frequency: pitch.frequency,
          points: points
        }
      })
    },
    graph() {
      return {
        domain: [0, 2 / this.pitches[0].frequency],
        range: [-1, 1],
      }
    },
    x() {
      return d3.scaleLinear()
        .domain(this.graph.domain)
        .range([0, 1605])
    },
    line() {
      return d3.line()
    }
  },
  methods: {
    d(f, t) {
      let omega = 2*Math.PI*f
      return Math.sin(omega*t)
    },
  },
  mounted() {
    console.log("hello")
    this.svg = d3.select(".chart").append("svg")
    if (this.pitches.length) {
      this.svg.append("g")
        .call(d3.axisBottom(this.x))
    }
  }
}
</script>

<style>
</style>