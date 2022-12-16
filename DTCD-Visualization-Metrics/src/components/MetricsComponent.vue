<template>
    <div
      class="metrics-container pa-3"
      style="min-height: 100%; padding: 15px;"
      :class="getClass"
      v-bind="$attrs"
    >
      <div class="header">
        <div>
          <span
            class="data-title"
            v-text="title"
          />
        </div>
      </div>
      <div
        v-if="error"
        :key="`error-message`"
        class="error-message"
      >
        {{ error }}
      </div>
      <div
        v-else
        class="content pt-3"
        :class="metricTemplateClass"
      >
        <template
          v-for="(metric) in metricsForRender"
        >
          <metric-component
            :metric="metric"
          />
        </template>
      </div>
    </div>
</template>

<script>
import MetricComponent from './MetricComponent';

export default {
  name: 'SingleValue',
  components: {MetricComponent},

  props: {
    dataRestFrom: {
      type: Array,
      required: true
    },
    template: {
      type: Number,
      required: true
    },
    metricsCount: {
      type: Number,
      required: true
    },
    title: {
      type: String,
      default: () => ''
    },
  },
  data: () => ({
    titleToken: '',
    metricList: [],
    isHeaderOpen: true,
    error: '',
  }),
  computed: {
    metricsForRender() {
      const elementsToShow = this.metricList.slice(0, this.metricsCount);
      if (elementsToShow.length !== elementsToShow.filter(Boolean).length) {
        this.setError('Допущен пропуск номеров в последовательности значений поля "_order"');
        return [];
      }
      this.setError('');

      return elementsToShow;
    },

    metricTemplateClass() {
      return `metric-${this.metricsCount} v-${this.template}`;
    },
    getClass() {
      return `header-active  ${this.header && 'is-header-open'}`;
    },
  },
  watch: {
    dataRestFrom: {
      handler(val) {
        if (val.length > 0) {
          this.setVisual([])
        }
      },
      deep: true,
    },
  },
  methods: {
    setError(err) {
      this.error = err;
    },
    setVisual(metricOptionsCurrent) {
      if (metricOptionsCurrent === undefined || metricOptionsCurrent.filter === undefined) {
        return;
      }
      const metricList = [];
      const metricOptions = [];
      this.error = '';
      structuredClone(this.dataRestFrom).forEach((data) => {
        const {
          metric, value, metadata, _order: sortOrder,
        } = data;
        const id = sortOrder;
          if (
            metricOptionsCurrent
            && metricOptionsCurrent.length !== metricOptionsCurrent.filter(Boolean).length
          ) {
            this.error = 'Допущен пропуск номеров в последовательности значений поля "_order"';
          }
          if (!sortOrder) {
            this.error = 'В запросе отсутствует обязательное поле "_order"';
            metricList.length = 0;
            metricOptions.length = 0;
            return;
          }
          if (this.error) return;

          let range;

          if (!metadata || typeof metadata !== 'string') {
            range = null;
          } else {
            range = metadata;
          }
          const startId = `${metric}_${id}`;

          const metricCurrent = metricOptionsCurrent?.find(
            (m) => m.startId === startId,
          );

          const defaultMetricOption = {
            title: metric || data.phase,
            ...metricCurrent,
            id: metricCurrent?.id || id,
            startId: metricCurrent?.startId || startId,
            metadata,
            color: metricCurrent?.color || 'main',
            icon: metricCurrent?.icon || 'no_icon',
            fontSize: metricCurrent?.fontSize || 54,
            fontWeight: metricCurrent?.fontWeight || 400,
            listOrder:
              metricCurrent?.listOrder === undefined
                ? sortOrder
                : metricCurrent?.listOrder,
          };
          metricList[(metricCurrent?.listOrder ?? sortOrder) - 1] = {
            value,
            ...defaultMetricOption,
          };

          metricOptions[(metricCurrent?.listOrder ?? sortOrder) - 1] = {
            ...defaultMetricOption,
            id,
            expanded: false,
            range,
          };

      });

      if (metricOptions.length > 0 && metricList.length > 0) {
        this.metricList = metricList;
      }
    },
  },
};
</script>

<style lang="sass" scoped>
@import '../scss/metrics'
</style>
