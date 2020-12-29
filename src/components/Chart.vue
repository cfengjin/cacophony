<template>
  <div class="container">
    <svg class="chart">
      <g class="bottom-axis" />
      <g class="left-axis" />
      <g class="lines" />
    </svg>
  </div>
</template>

<script>
import * as d3 from "d3"

export default {
  props: ['pitches', 'showFundamentals', 'showInterference'],
  data() {
    return {
      chart: {
        height: 0,
        width: 0,
      },
      period: 1 / 27.5
    }
  },
  computed: {
    data() {
      let delta = .0001;

      let fundamentals = this.pitches.map(pitch => {
        let pts = [];
        for (let i = 0; i < 1/delta; ++i) {
          let t = delta * i * this.period
          pts.push({
            time: t,
            displacement: this.d(pitch.frequency, t)
          })
        }

        return {
          class: pitch.class,
          frequency: pitch.frequency,
          points: pts
        }
      })

      if (fundamentals.length > 1 && this.showInterference) {
        let pts = [];
        for (let i = 0; i < 1/delta; ++i) {
          let t = delta * i * this.period
          pts.push({
            time: t,
            displacement: fundamentals.reduce((acc, obj) => acc + obj.points[i].displacement, 0),
          })
        }

        let interference = {
          class: "Chord",
          points: pts
        }

        if (this.showFundamentals) {
          return fundamentals.concat(interference)
        }
        return [interference]
      }

      if (this.showFundamentals) {
        return fundamentals
      }
      return []
    },
    x() {
      return d3.scaleLinear()
        .domain([0, this.period])
        .range([48, this.chart.width])
    },
    y() {
      return d3.scaleLinear()
        .domain([this.showInterference ? -this.pitches.length : -1, this.showInterference ? this.pitches.length : 1])
        .range([this.chart.height - 24, 24])
    },
    line() {
      return d3.line()
        .x(d => this.x(d.time))
        .y(d => this.y(d.displacement))
    },
    color() {
      return d3.scaleOrdinal()
        .domain(['A', 'A#', 'B', 'C', 'C#', 'D', 'D#', 'E', 'F', 'F#', 'G', 'G#', 'Chord'])
        .range(['#4e79a7', '#f28e2c', '#e15759', '#76b7b2', '#59a14f', '#edc949', '#af7aa1', '#ff9da7', '#9c755f', '#bab0ab', '#888888', 'whitesmoke', '#ffffff'])
    }
  },
  methods: {
    d(f, t) {
      let omega = 2 * Math.PI * f
      return Math.sin(omega * t)
    },
    renderChart() {
      this.chart.width = document.querySelector(".container").clientWidth
      this.chart.height = document.querySelector(".container").clientHeight

      d3.select(".chart")
        .attr("display", "block")
        .attr("viewBox", [0, 0, this.chart.width, this.chart.height])

      d3.select(".bottom-axis")
        .attr("transform", `translate(0, ${this.chart.height / 2})`)
        .attr("stroke", "whitesmoke")
        .call(d3.axisBottom(this.x))
      d3.select(".left-axis")
        .attr("transform", `translate(48, 0)`)
        .attr("stroke", "whitesmoke")
        .call(d3.axisLeft(this.y))
      d3.selectAll(".domain")
        .attr("stroke", "whitesmoke")

      d3.select(".lines")
        .attr("display", "initial")
        .attr("fill", "none")
        .attr("stroke-linejoin", "round")
        .attr("stroke-linecap", "round")
        .selectAll("path")
        .data(this.data)
        .join("path")
        .attr("d", d => this.line(d.points))
        .attr("stroke", d => this.color(d.class))
        .attr("stroke-width", d => d.class == "Chord" ? 3 : 1.5)
    }
  },
  mounted() {
    this.renderChart()
    window.addEventListener('resize', this.renderChart);
  },
  beforeUpdate() {
    this.renderChart()
  }
}
</script>

<style>
.container {
  overflow: hidden;
  flex-grow: 1;
  background-color: #1f1f1f;
}
</style>