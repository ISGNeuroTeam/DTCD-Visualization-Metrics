<template>
  <div class="VisualizationMetrics">
   <metrics-component
     :dataRestFrom="dataset"
     :template="template"
     :metricsCount="metricsCount"
     :title="title"
   />
  </div>
</template>

<script>

import MetricsComponent from './components/MetricsComponent';

export default {
  name: 'Metrics',
  components: {
    MetricsComponent
  },
  data: () => ({
    dataset: [],
    template: 1,
    metricsCount: 1,
    title: '',
    titleFromConfig: ''
  }),
  methods: {
    validateTemplate({metricsCount, template}){
      if (metricsCount < 0 && metricsCount > 6 || template < 1) {
        return false
      }
      const maxTemplatesForMetricsCount = {
        1:1,
        2:2,
        3:6,
        4:7,
        5:4,
        6:2,
      }

      return maxTemplatesForMetricsCount[metricsCount] >= template
    },
    setTitle(title = '') {
      this.titleFromConfig = title;
    },
    setDataset(data = []) {
      let isConfigInDataset = false
      this.dataset = data.reduce((acc, item) => {
        if (!!(item._template || item._metricsCount || item._header)) {
          isConfigInDataset= true
          const isValidTemplate = this.validateTemplate({
            template: item._template,
            metricsCount: item._metricsCount,
          })

          this.template = isValidTemplate ? item?._template : 1
          this.metricsCount = isValidTemplate ? item?._metricsCount : 1

          this.title = this.titleFromConfig || item?._header || ''
        }
        if (!(item._template || item._metricsCount || item._header)) {
          return [...acc, item]
        }

        return acc
      },[])
      if (!isConfigInDataset) {
        this.clearConfig()
      }
    },
    clearConfig() {
      this.template = 1
      this.metricsCount = 1
      this.title = this.titleFromConfig || ''
    }
  },
};
</script>

<style lang="scss" scoped>
.VisualizationMetrics {
  width: 100%;
  height: 100%;
  .title {
    font-weight: 600;
    font-size: 15px;
    line-height: 18px;
    padding: 10px;
    color: var(--text_main);
  }
}

</style>
