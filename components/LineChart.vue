<template>
  <div class="chart-container relative p-8">
    <canvas
      id="myChart"
      ref="chart"
      height="400"
    />
  </div>
</template>

<script>
import 'chart.js';

export default {
  props: {
    datasets: { type: Array, required: true },
    labels: { type: Array, required: true },
  },
  data: () => ({
    chart: null,
  }),
  computed: {
    numDatasets() {
      return this.datasets.length;
    },
  },
  watch: {
    numDatasets() {
      this.update();
    },
  },
  mounted() {
    this.chart = new window.Chart(this.$refs.chart, {
      type: 'line',
      data: {
        labels: this.labels,
        datasets: this.datasets,
      },
      options: {
        maintainAspectRatio: false,
        legend: {
          display: false,
        },
        tooltips: {
          enabled: true,
          displayColors: false,
          callbacks: {
            label(tooltipItem, data) {
              return data.datasets[tooltipItem.datasetIndex].label || '';
            },
            title() {
              return '';
            },
          },
        },
        scales: {
          yAxes: [{
            type: 'linear',
            scaleLabel: {
              display: true,
              labelString: 'Cases per million in population',
            },
            gridLines: {
              drawBorder: false,
            },
          }],
          xAxes: [{
            scaleLabel: {
              display: true,
              labelString: 'Days since reaching 1 per million in population',
            },
            gridLines: {
              display: false,
            },
          }],
        },
      },
    });
  },
  methods: {
    update() {
      this.chart.data.labels = this.labels;
      this.chart.data.datasets = this.datasets;
      this.chart.update();
    },
  },
};
</script>

<style scoped>
canvas {
  width: 100%;
}

.chart-container {
  width: 100%;
  min-width: 360px;
  height: 50vh;
}
</style>
