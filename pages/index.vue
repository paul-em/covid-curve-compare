<template>
  <section>
    <github-corner url="https://github.com/paul-em/covid-curve-compare"/>
    <h1 class="px-16 pt-16 max-w-xl">
      Population adjusted curves of active Covid-19 cases
      starting from day of reaching 1 case per million
    </h1>
    <p class="px-16 py-2">
      Updated once per day <a
        class="text-blue"
        target="_blank"
        href="https://github.com/CSSEGISandData/COVID-19">by John Hopkins University</a>
    </p>
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
    const [confirmed, deaths, recovered] = await Promise.all([
      $axios.get('https://covid19.cors-everywhere.workers.dev?metric=confirmed'),
      $axios.get('https://covid19.cors-everywhere.workers.dev?metric=deaths'),
      $axios.get('https://covid19.cors-everywhere.workers.dev?metric=recovered'),
    ]);
    const active = {};
    Object.keys(confirmed.data).forEach((area) => {
      active[area] = confirmed.data[area].map((value, index) => value
          - ((deaths.data[area] || [])[index] + (recovered.data[area] || [])[index]));
    });
    return {
      confirmed: active,
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
    startAt: 1,
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
        borderColor: this.$color.rgba(area, 0.5),
        hoverBorderColor: this.$color.rgba(area, 1),
        pointRadius: 0,
        pointHoverRadius: 0,
        pointHitRadius: 20,
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
        if (startAdjusted.length || val >= this.startAt) {
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
