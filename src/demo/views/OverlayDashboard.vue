<template>
  <div class="db-layers">
    <div class="db-layer">
      <db-dashboard v-if="ready" :dbspec="dbspec" :dbdata="dbdata" :dark="isDark" @db-event="handleDbEvent"> </db-dashboard>
    </div>
    <div class="db-layer">
      <db-dashboard :dbspec="dbspec2" :dbdata="dbdata2" :dark="isDark" class="db-transparent"> </db-dashboard>
    </div>
  </div>
</template>

<script>
import DbDashboard from '@/components/dashboard/DbDashboard.vue';
import DbData from '../../components/dbdata';
import { demodashboard } from '../mixins/demodashboard';
import pathOr from 'ramda/es/pathOr';
import perspective from '@finos/perspective';
import moment from 'moment';
import flareData from '@/demo/data/flare.json';

export default {
  name: 'SalesSunburstDashboard',
  components: {
    DbDashboard
  },
  mixins: [demodashboard],
  data() {
    return {
      dbdata: new DbData(),
      dbspec: {
        layout: {
          type: 'grid',
          size: 12
        },
        colorScheme: 'DivergingRdYlBu',
        widgets: [
          {
            id: 'w2',
            type: 'DbDygraphsBar',
            cspan: 12,
            height: 800,
            properties: {
              options: {
                stackedGraph: false,
                labels: ['Date', 'Visit', 'Conversion'],
                legend: 'never',
                drawAxesAtZero: false,
                series: {
                  Visit: { axis: 'y2' },
                  Conversion: { axis: 'y2' }
                },
                axes: {
                  y: {
                    drawAxis: true,
                    drawGrid: true,
                    axisLabelWidth: 0
                  }
                }
              }
            }
          }
        ]
      },
      dbdata2: new DbData(),
      dbspec2: {
        layout: {
          type: 'grid',
          size: 12
        },
        colorScheme: 'DivergingRdYlBu',
        widgets: [
          {
            id: 'wA',
            type: 'DbMarkup',
            cspan: 12,
            height: 140,
            properties: {
              markup: '<div class="overlay-title">CONVERSIONS TREND</div><div class="overlay-subtitle">Site visits and conversion rate with geo and source splits</div>'
            }
          },
          {
            id: 'wA',
            type: 'DbChartjsDoughnut',
            cspan: 2,
            height: 200,
          },
          {
            id: 'wB',
            type: 'DbChartjsDoughnut',
            cspan: 2,
            height: 200,
            properties: {
              colorScheme: 'DivergingPiYG',
            }
          },
          {
            id: 'wAN',
            type: 'DbNone',
            cspan: 8
          },
          {
            id: 'wC',
            type: 'DbChartjsDoughnut',
            cspan: 2,
            height: 200,
            properties: {
              colorScheme: 'DivergingPiYG',
            }
          },
          {
            type: 'DbNone',
            cspan: 10
          }
        ]
      },
      ready: false,
      startTimestamp: null,
      endTimestamp: null,
      worker: null,
      table: null,
      view: null
    };
  },
  async mounted() {
    this.worker = perspective.worker();
    this.$store.dispatch('setDashboardSpec', { spec: JSON.stringify(this.dbspec, null, '\t') });
    await this.initialize();
  },
  methods: {
    initialize: async function() {
      await this.showData();
    },

    showData: async function() {
      let dthData = [];
      let sTS = Date.now() - 100 * 3600 * 1000;

      for (let i = 0; i < 100; i++) {
        let cTs = sTS + i * 3600 * 1000;
        let d = new Date(cTs);
        let r = Math.random();
        let e = Math.random();
        let ss = ((Math.sin(i / 3) + 3) * 8 - Math.random() * 3) * i * i;
        let se = ((Math.sin(i / 3) + 2) * 4 - Math.random() * 2) * i * i;
        dthData.push([d, ss, se]);
      }

      this.dbdata.setWData('w2', {
        data: dthData
      });

      this.dbdata2.setWData('wA', {
        data: {
          labels: ['Visits', 'Conversions'],
          datasets: [{ data: [276000, 109000] }]
        }
      });

      this.dbdata2.setWData('wB', {
        data: {
          labels: ['Organic', 'Ads', 'Social'],
          datasets: [{ data: [120000, 80000, 98000] }]
        }
      });

      this.dbdata2.setWData('wC', {
        data: {
          labels: ['AMER', 'EMEA', 'APAC'],
          datasets: [{ data: [300000,180000,70000] }]
        }
      });

      this.ready = true;
    },

    getRand: function(max) {
      return Math.floor(Math.random() * Math.floor(max));
    },
    handleDbEvent(event) {
      switch (event.type) {
        case 'zoom': {
          let from = pathOr(null, ['minDate'], event);
          let to = pathOr(null, ['maxDate'], event);
          if (from && to) {
            this.startTimestamp = from;
            this.endTimestamp = to;
            this.showData();
            //let mFrom = moment(from).format('YYYY/MM/DD HH:mm:ss');
            //let mTo = moment(to).format('YYYY/MM/DD HH:mm:ss');
            // reset local time range settings, so it'll be re-calculated
            //this.setRange({ from: from, to: to, title: `${mFrom} - ${mTo}` });
            // Query will be triggered by time range change watch
          }
          break;
        }
      }
    }
  }
};
</script>
<style lang="scss">
.db-layers {
  width: 90%;
  height: 820px;
  position: relative;
  margin-left: 50px;

  .db-layer {
    position: absolute;
    width: 100%;
    top: 0px;
    left: 0px;
  }
}

.overlay-title {
  font-size: 120px;
  position: absolute;
  top: -35px;
  left: 0;
  color: #D07345;
  opacity: 0.8;
}
.overlay-subtitle {
  font-size: 40px;
  position: absolute;
  top: 100px;
  left: 10px;
  color: #D07345;
  opacity: 0.6;
}
</style>
