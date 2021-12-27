<template>
  <div class="chart-wrapper">
    <div class="chart-header">
      <div class="chart-title">{{ selected.name }} - USD</div>
      <div class="chart-close" @click="closeChart">х</div>
    </div>
    <div class="chart" ref="graph">
      <div class="chart-item"
           v-for="(bar, idx) in normalizedGraph"
           :key="idx"
           :style="{ height: `${bar}%`, width: graphElementWidth + 'px' }"
           :title="bar"
           ref="graphElement"
      ></div>
    </div>
    <label class="change-column-width">
      <span>Изменить ширину столбца</span>
      <input type="text"
             v-model="graphElementWidth"
             title="Ширина столбца"
      >
    </label>
  </div>
</template>

<script>
export default {
  props: ['selected', 'graph'],
  data() {
    return {
      maxGraphElements: 1,
      graphElementWidth: 5,
      selectedTicker: this.selected
    }
  },
  created() {
    const graphEl = localStorage.getItem('cryptonomicon-column')
    this.graphElementWidth = JSON.parse(graphEl)
    // this.calculateMaxGraphElements()
  },
  methods: {
    closeChart() {
      this.selectedTicker = null
      this.$emit('close-chart', this.selectedTicker)
    },
    calculateMaxGraphElements() {
      if (!this.$refs.graph) {
        return;
      }
      this.maxGraphElements = this.$refs.graph.clientWidth / this.graphElementWidth
      this.$emit('max-columns', this.maxGraphElements)
      // console.log(this.maxGraphElements);
    },

  },
  computed: {
    normalizedGraph() {
      const maxValue = Math.max(...this.graph)
      const minValue = Math.min(...this.graph)

      if (maxValue === minValue) {
        return this.graph.map(() => 50)
      }

      return this.graph.map(
        price => 5 + ((price - minValue) * 95 / (maxValue - minValue))
      )
    },
  },
  watch: {
    graphElementWidth() {
      localStorage.setItem('cryptonomicon-column', JSON.stringify(this.graphElementWidth))
      this.calculateMaxGraphElements();

    },
    graph() {
      this.calculateMaxGraphElements();
    }
  }
}
</script>

<style scoped>

</style>