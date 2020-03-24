<template>
  <section>
    <github-corner url="https://github.com/paul-em/covid-curve-compare"/>
    <h1 class="p-16 max-w-xl">
      Population adjusted curves of confirmed Covid-19 cases
      starting from day of reaching 1 case per million
    </h1>
    <area-select
      v-model="selectedAreas"
      :areas="areas"
    />
    <line-chart
      :labels="dates"
      :datasets="datasets"
    />
  </section>
</template>

<script>
import AreaSelect from '../components/AreaSelect.vue';
import LineChart from '../components/LineChart.vue';
import GithubCorner from '../components/GithubCorner.vue';
import populations from '../assets/populations';

export default {
  async asyncData({ $axios }) {
    const re = await $axios.get('https://covid19.cors-everywhere.workers.dev');
    return {
      confirmed: re.data,
    };
  },
  components: {
    AreaSelect,
    LineChart,
    GithubCorner,
  },
  data: () => ({
    selectedAreas: ['Austria', 'Germany', 'Italy', 'Spain', 'Korea', 'Hubei, China', 'US'],
    confirmed: {},
  }),
  computed: {
    areas() {
      return Object
        .keys(this.confirmed)
        .filter(area => populations[area])
        .filter(area => this.confirmed[area].find(i => i > 200));
    },
    missingPopulations() {
      return Object.keys(this.confirmed)
        .filter(area => !populations[area] && this.confirmed[area].find(i => i > 200));
    },
    dates() {
      const dates = [];
      for (let i = 0; i < this.confirmed[this.areas[0]].length; i += 1) {
        dates.push(i);
      }
      return dates;
    },
    datasets() {
      return this.selectedAreas.map(area => ({
        label: area,
        data: this.getAdjustedData(area),
        backgroundColor: 'rgba(0, 0, 0, 0)',
        borderColor: this.$color.rgba(area, 0.8),
        pointRadius: 0,
      }));
    },
  },
  watch: {
    selectedAreas() {
      this.$router.push({
        query: {
          shown: this.selectedAreas,
        },
      });
    },
  },
  mounted() {
    if (this.$route.query.shown) {
      this.selectedAreas = this.$route.query.shown;
    }
  },
  methods: {
    getAdjustedData(area) {
      // map to population
      const perMillionTimline = this.confirmed[area]
        .map(val => Math.round((val / populations[area]) * 10000000) / 10);
      // start when per million hits 1
      const startAdjusted = [];
      perMillionTimline.forEach((val) => {
        if (startAdjusted.length || val >= 1) {
          startAdjusted.push(val);
        }
      });
      return startAdjusted;
    },
  },
};
</script>

<style>
</style>
