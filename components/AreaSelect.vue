<template>
  <div class="flex flex-col align-center p-16 area-select">
    <div class="flex-1"/>
    <div>
      <span
        v-for="item in value"
        :key="item"
        :style="{ 'border-color': $color.hex(item) }"
        class="border-b-2 mr-4 my-2 inline-block cursor-pointer"
        @click="unselect(item)"
      >
        {{ item }}
      </span>
      <div class="inline-block min-w-200">
        <v-select
          :options="notSelectedAreas"
          :value="null"
          placeholder="Add another"
          @input="select"/>
      </div>
    </div>
    <div class="flex-1"/>
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
.area-select {
  min-height: 100px;
}

.vs__dropdown-toggle {
  border: 0;
}
</style>
