<template>
  <chart
    v-if="ready"
    :options="options"
    v-bind="config"
    :style="{
      width: (context && config && config.width) ? config.width + 'px' : (width) ? width + 'px' : 'auto',
      height: (context && config && config.height) ? config.height + 'px' : (height) ? height + 'px' : '400px',
      ...(config) ? config.style : {} }"
    :theme="$f7.data.themeOptions.dark === 'dark' ? 'dark' : undefined" autoresize></chart>
</template>

<script>
import mixin from '../widget-mixin'
import 'echarts/lib/chart/gauge'

import ECharts from 'vue-echarts/components/ECharts'
import { OhChartDefinition } from '@/assets/definitions/widgets/chart/page'

export default {
  mixins: [mixin],
  components: {
    'chart': ECharts
  },
  widget: OhChartDefinition,
  computed: {
    options () {
      const chartConfig = this.config.options || {}
      return {
        ...chartConfig,
        series: this.series
      }
    },
    series () {
      if (!this.context.component.slots || !this.context.component.slots.series) return undefined
      return this.context.component.slots.series.map((s) => this.getSerie(s))
    }
  },
  data () {
    return {
      ready: false
    }
  },
  mounted () {
    this.ready = true
  },
  methods: {
    serieConfig (component) {
      const sourceConfig = component.config
      let evalConfig = {}
      if (component.config) {
        if (typeof component.config !== 'object') return {}
        for (const key in component.config) {
          if (key === 'visible' || key === 'visibleTo') continue
          this.$set(evalConfig, key, this.evaluateExpression(key, sourceConfig[key]))
        }
      }
      return evalConfig
    },
    value (component) {
      const item = this.evaluateExpression("item", component.config.item)
      const value = this.context.store[item].state
      return parseFloat(value)
    },
    getSerie (component) {
      const config = this.serieConfig(component) 
      return {
        ...config,
        data: [{ value: this.value(component), name: this.evaluateExpression("name", component.config.name) }]
      }
    }
  }
}
</script>
