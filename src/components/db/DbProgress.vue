/* DashBlocks: Number widget */
<template>
  <div class="db-progress">
    <div class="db-progress-background">
      <db-sparkline :_updated="_updated" :data="trend" :height="60"></db-sparkline>
      <db-linear-progress :_updated="_updated" :value="value" :height="15"></db-linear-progress>
    </div>
    <div class="db-rsb db-progress-front">
      <div style="border: 1px solid #8e24aa;">
        <div class="text-lg db-txt-faded">{{ title }}</div>
        <div class="text-sm db-txt-faded">{{ subtitle }}</div>
      </div>
      <div class="text-xl db-txt-highlight" style="text-align: right">{{ formattedValue }}</div>
    </div>
  </div>

  <!--<div class="db-number">
    <div class="db-n-content">
      <div class="db-n-layer">
      </div>
      <div class="db-n-layer" style="text-align: right;">
        <div style="float: right;">
          <div class="db-n-value db-txt-highlight">{{ formattedValue }}</div>
        </div>
      </div>
      <div class="db-n-main">
        <div class="db-n-hdr">
          <span class="text-md db-txt-faded">{{ title }}</span>
        </div>
        <div>
          <div class="text-sm db-txt-faded">{{ subtitle }}</div>
        </div>
        <db-linear-progress :_updated="_updated" :value="value" :height="20"></db-linear-progress>
      </div>
    </div>
  </div>-->
</template>

<script>
import DbSparkline from './DbSparkline';
import DbLinearProgress from './DbLinearProgress';
import DbEasyPie from './DbEasyPie';
import { dbStdProps } from '../mixins/dbstdprops';
import { sprintf } from 'sprintf-js';
export default {
  name: 'DbNumber',
  components: {
    DbSparkline,
    DbLinearProgress
  },
  mixins: [dbStdProps],
  props: {
    value: {
      type: Number,
      default: 0
    },
    format: {
      type: String,
      default: '%d %%'
    },
    total: {
      type: Number,
      default: 0
    },
    trend: {
      type: Array,
      default: () => []
    },
    trendMax: {
      type: Number,
      default: null
    },
    ranges: {
      type: Array,
      default: () => []
    },
    percentRanges: {
      type: Array,
      default: () => []
    },
    icon: {
      type: String,
      default: ''
    },
    title: String,
    subtitle: String,
    qualifier: {
      type: String,
      default: ''
    },
    footer: {
      type: String,
      default: ''
    },
    // TODO ???
    badge: {
      type: String,
      default: ''
    }
  },
  data() {
    return {
      // TODO Better gradient for light and dark
      trendGradient: ['#DCEDC8', '#FFF9C4', '#FFCCBC']
    };
  },
  /*
  watch: {
    dark: function() {
      this.scheduleUpdate(true);
    }
  },
  */
  computed: {
    hasTrend() {
      return this.trend && Array.isArray(this.trend) && this.trend.length > 0;
    },
    hasRanges() {
      return this.ranges && Array.isArray(this.ranges) && this.ranges.length == 2;
    },
    hasIcon() {
      // TODO if we have trend, and don't have explicit icon, show trend up / down
      return this.icon !== '';
    },
    hasPct() {
      return this.total > 0;
    },
    iconClass() {
      return this.icon + ' db-n-icon ' + this.getRangeClass();
    },
    formattedValue() {
      return sprintf(this.format, this.value, this.qualifier);
    },
    percentValue() {
      if (this.total > 0) {
        return (this.value / this.total) * 100;
      } else {
        return 0;
      }
    }
  },
  methods: {
    getRangeClass() {
      if (!this.hasRanges) {
        return 'db-txt-faded';
      }
      if (this.value <= this.ranges[0]) {
        return 'text-success';
      } else if (this.value <= this.ranges[1]) {
        return 'text-warning';
      } else {
        return 'text-alarm';
      }
    }
  }
};
</script>
<style lang="scss">
.db-progress {
  width: 100%;
  height: 100%;
  min-height: 100px;
  position: relative;
  border: 1px solid saddlebrown;

  .db-progress-front {
    height: 80%;
    min-height: 80px;
    align-items: center;
  }

  .db-progress-background {
    position: absolute;
    left: 0px;
    bottom: 0px;
    border: 1px solid #00b8d4;
    width: 100%;
    height: 90%;
    display: grid;
    /* align-items: center; */
  }
}
</style>
