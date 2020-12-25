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
  props: ['pitches'],
  data() {
    return {
      chart: {
        height: 400,
        width: window.innerWidth,
      }
    }
  },
  computed: {
    data() {
      let delta = .001;

      return this.pitches.map(pitch => {
        let points = [];

        for (let i = 0; i < 1/delta; ++i) {
          let period = 1 / this.pitches[0].frequency
          let t = delta * i * 4 * period
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
    x() {
      return d3.scaleLinear()
        .domain([0, 4 / this.pitches[0].frequency])
        .range([100, this.chart.width - 100])
    },
    y() {
      return d3.scaleLinear()
        .domain([-1, 1])
        .range([50, this.chart.height - 50])
    },
    line() {
      return d3.line()
        .x(d => this.x(d.time))
        .y(d => this.y(d.displacement))
    }
  },
  methods: {
    d(f, t) {
      let omega = 2 * Math.PI * f
      return Math.sin(omega * t)
    },
    renderChart() {
      this.chart.width = window.innerWidth

      if (this.pitches.length) {
        d3.select(".chart")
          .attr("display", "block")
          .attr("viewBox", [0, 0, this.chart.width, this.chart.height])

        d3.select(".bottom-axis")
          .attr("transform", `translate(0, ${this.chart.height / 2})`)
          .call(d3.axisBottom(this.x))
        d3.select(".left-axis")
          .attr("transform", `translate(100, 0)`)
          .call(d3.axisLeft(this.y))
        
        d3.select(".lines")
          .attr("fill", "none")
          .attr("stroke", "black")
          .attr("stroke-width", 2)
          .attr("stroke-linejoin", "round")
          .attr("stroke-linecap", "round")
          .selectAll("path")
          .data(this.data)
          .join("path")
          .attr("d", d => this.line(d.points));
      } else {
        d3.select(".chart")
          .attr("display", "none")
      }
    }
  },
  mounted() {
    window.addEventListener('resize', this.renderChart);
    this.renderChart()
  },
  beforeUpdate() {
    this.renderChart()
  }
}
</script>

<style>
.container {
  height: 400px;
}
</style>