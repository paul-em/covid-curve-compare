<template>
  <div class="p-8">
    <span
      v-for="item in value"
      :key="item"
      :style="{ 'border-color': $color.hex(item) }"
      class="border-b-2 mr-2"
      @click="unselect(item)"
    >
      {{ item }}
    </span>
    <div class="inline-block min-w-100">
      <v-select
        :options="notSelectedAreas"
        :value="null"
        @input="select"/>
    </div>
  </div>
</template>

<script>
import vSelect from 'vue-select';
import 'vue-select/dist/vue-select.css';

export default {
  components: {
    vSelect,
  },
  model: {
    prop: 'value',
    event: 'input',
  },
  props: {
    areas: { type: Array, required: true },
    value: { type: Array, required: true },
  },
  computed: {
    notSelectedAreas() {
      return this.areas.filter(area => !this.value.includes(area));
    },
  },
  methods: {
    select(area) {
      this.$emit('input', [
        ...this.value,
        area,
      ]);
    },
    unselect(area) {
      this.$emit('input', [...this.value].filter(a => a !== area));
    },
  },
};
</script>

<style>
</style>
