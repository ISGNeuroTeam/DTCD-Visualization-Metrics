<template>
  <div
    :key="`metric-${metric.startId}`"
    class="item"
    :style="{ gridArea: `item-${metric.listOrder}` }"
  >
    <span class="metric-title">
    <span
      v-show="metric.icon !== 'no_icon'"
      class="icon"
      v-html="getIconSvgByID"
    />
    <v-icon
      v-show="metric.icon === 'no_icon'"
      :style="`color: ${getColor} !important;`"
      v-text="getIcon"
    />
    <span
      class="title-text"
      v-text="metric.title"
    />
    </span>
    <span
      v-if="metric.value"
      class="metric-value"
      :class="`color-${metric.color}`"
      style="
               color: #5980f8;
               font-size: 54px;
               font-weight: 400;
             "
      :style="{
        ...getColor,
        ...getFontStyle,
        display: metric.value && metric.value.toString().split(',').length > 1
        ? 'flex'
        : 'block',
        }
      "
    >
      <span
        v-for="(value, inx) in metric.value.toString().split(',')"
        :key="inx"
      >
        {{ value + (inx !== metric.value.toString().split(',').length -1
        ? ', '
        : '') | filterNumber({numberPerDigit: false}) }}
      </span>
    </span>
  </div>
</template>
<script>
export default {
  name: 'MetricComponent',
  filters: {
    filterNumber: (value, options) => {
      if (!value) return '';
      if (!options?.numberPerDigit) return value;
      const numberOfDigits = (num) => Number(num).toLocaleString('ru-RU');
      if (value.match(/^-?\d+$/)) {
        // number
        return numberOfDigits(value);
      }
      if (value.match(/^\d+\.\d+$/)) {
        // float
        const filteredValue = numberOfDigits(value.split('.')[0]);
        return `${filteredValue}.${value.split('.')[1]}`;
      }
      // not number
      return value;
    },
  },
  props: {
    metric: {
      type: Object,
      required: true,
    },
  },
  computed: {
    getIcon() {
      if (!this.metric.metadata) {
        return ;
      }
      const ranges = JSON.parse(this.metric.metadata.replaceAll("'", '"'));
      if (ranges.icon) {
        // return iconlist.find((icon) => ranges.icon === icon);
        return [];
      }
      return '';
    },
    getFontStyle() {
      if (!this.metric.metadata) {
        return
      }

      const ranges = JSON.parse(this.metric.metadata.replaceAll("'", '"'));

      if (!ranges.font) {
        return
      }

      return {
        fontSize: `${ranges.font?.fontSize}px`,
        fontWeight: ranges.font?.fontWeight,
      };
    },
    getColor() {
      if (!this.metric.metadata) {
        return
      }
      let color;
      const ranges = JSON.parse(this.metric.metadata.replaceAll("'", '"'));
      if (!ranges.range) {
        Object.keys(ranges).forEach((key) => {
          ranges[key] = `${ranges[key]}`.split(':');
        });
        if (!Number.isNaN(+this.metric.value)) {
          const val = Number(this.metric.value);
          color = Object.keys(ranges).reduce((resultColor, color) => {
            const range = ranges[color]
            const colorRange = range[0]?
              range[0] < range[1]
                ? val >= range[0] && val <= range[1]
                : val >= range[0]
              : val <= range[1];
            return colorRange ? color: resultColor
          },null)
        } else {
          const val = this.metric.value;
          color = Object.keys(ranges).reduce((resultColor, color) => {
            return ranges[color].includes(val) ? color : resultColor
          },null)
        }
      } else if (ranges?.range.length > 0) {
        ranges.range.forEach((item, index) => {
          ranges.range[index] = item.split(':');
        });
        if (ranges.range.length === ranges.colors.length) {

          if (!Number.isNaN(+this.metric.value)) {
            const val = Number(this.metric.value);
            color = ranges.colors.reduce((resultColor, color, index) => {
              const range = ranges.range[index]
              const colorRange = range[0]?
                range[0] < range[1]
                  ? val >= range[0] && val <= range[1]
                  : val >= range[0]
                : val <= range[1];

              return colorRange ? color: resultColor

            }, null);

          } else {
            const val = this.metric.value;
            color = ranges.colors.reduce((resultColor, color, index) => {
              const range = ranges.range[index]
              if (range.includes(val)) {
                return color
              }
              return resultColor

            }, null);
          }
        }
      }

      return{color}
    },
  }
};
</script>
<style lang="sass" scoped>
@import '../scss/metrics'
</style>