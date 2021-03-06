<template>
  <div>
    <q-toolbar>
      <q-select dense outlined v-model="dataSet" :options="dataOptions" label="Data" emit-value map-options />
      <q-select dense outlined v-model="colorInterpolator" toggle-color="primary" label="Colors" :options="colorOptions" emit-value map-options />
      <q-toggle v-model="lineEnabled" icon="waves">
        <q-tooltip anchor="bottom right" self="center middle">Enable Outline</q-tooltip>
      </q-toggle>
    </q-toolbar>
    <db-dashboard v-if="ready" :dbspec="dbspec" :dbdata="dbdata" :dark="isDark"> </db-dashboard>
  </div>
</template>

<script>
import { DbData, DbDashboard } from 'dashblocks';
import { demodashboard } from '@/demo/mixins/demodashboard';
import ridgedata from '../data/ridgedata.json';
import ridgemapdata from '../data/ridgemapdata.json';
import apireqdata from '../data/series_rate_api_req_count.json';
import jvmdata from '../data/series_test_jvm_memory_used_bytes.json';
//import metrics3 from '../data/metrics3.json';

export default {
  name: 'SparkLineMap',
  components: {
    DbDashboard
  },
  mixins: [demodashboard],
  data() {
    return {
      lineEnabled: false,
      dataOptions: [
        { label: 'Metrics (1000+)', value: 't2' },
        { label: 'Metrics (160)', value: 'm1' },
        { label: 'Metrics 2 (160)', value: 'm2' },
        { label: 'Metrics 3 (42)', value: 'm3' },
        { label: 'Ridge 1', value: 'r1' },
        { label: 'Ridge 2', value: 'r2' },
        { label: 'Small Test (10)', value: 't1' }
      ],
      colorOptions: [
        { label: 'Blues', value: 'interpolateBlues' },
        { label: 'Reds', value: 'interpolateReds' },
        { label: 'Oranges', value: 'interpolateOranges' },
        { label: 'Greens', value: 'interpolateGreens' },
        { label: 'Purples', value: 'interpolatePurples' },
        { label: 'Greys', value: 'interpolateGreys' }
      ],
      dataSet: 'm2',
      colorInterpolator: 'interpolateBlues',
      dbdata: new DbData(),
      dbspec: {
        layout: {
          type: 'grid'
        },
        widgets: [
          {
            id: 'wSLM',
            type: 'DbSparkLineMap',
            cspan: 16,
            height: 600,
            properties: {
              colorInterpolateScheme: 'interpolateOranges'
            }
          }
        ]
      },
      ready: false,
      testData1K: null,
      testData: [
        [0, 0, 1, 1, 2, 2, 3, 4, 5, 6, 5, 4, 3, 2, 1, 0],
        [0, 0, 0, 0, 1, 1, 2, 3, 8, 4, 4, 3, 2, 1, 1, 0],
        [0, 3, 4, 5, 1, 1, 2, 2, 2, 6, 6, 5, 4, 2, 2, 2],
        [0, 3, 4, 5, 1, 1, 2, 2, 2, 6, 6, 5, 4, 2, 2, 2],
        [0, 3, 4, 5, 1, 1, 2, 2, 2, 6, 6, 5, 4, 2, 2, 2],
        [0, 3, 4, 5, 1, 1, 2, 2, 2, 6, 6, 5, 4, 2, 2, 2],
        [0, 3, 4, 5, 8, 1, 2, 2, 2, 6, 6, 5, 4, 2, 2, 2],
        [0, 3, 4, 5, 1, 1, 2, 2, 2, 6, 6, 5, 4, 2, 2, 2],
        [0, 3, 4, 5, 1, 1, 2, 2, 2, 6, 6, 5, 4, 2, 2, 2],
        [0, 3, 4, 5, 1, 1, 2, 2, 2, 6, 6, 5, 4, 2, 2, 2]
      ]
    };
  },
  watch: {
    lineEnabled() {
      this.$nextTick(function() {
        this.setData();
      });
    },
    dataSet() {
      this.$nextTick(function() {
        this.setData();
      });
    },
    colorInterpolator() {
      this.$nextTick(function() {
        this.setData();
      });
    }
  },
  async mounted() {
    await this.$store.dispatch('setDashboardSpec', { spec: JSON.stringify(this.dbspec, null, '\t') });
    this.dataSet = this.$route.query.ds || 't2';
    this.colorInterpolator = this.$route.query.colors || 'interpolateBlues';
    await this.initialize();
    this.ready = true;
  },
  methods: {
    initialize: async function() {
      let data1K = [];
      for (let s of apireqdata) {
        for (let i = 0; i < 7; i++) {
          data1K.push(s);
        }
      }
      this.testData1K = Object.freeze(data1K);
      this.setData();
    },

    setData: function() {
      let line = this.lineEnabled;
      let ds = null;
      switch (this.dataSet) {
        case 'm1': {
          ds = apireqdata;
          break;
        }
        case 'm2': {
          ds = jvmdata;
          break;
        }
        case 'm3': {
          ds = jvmdata; //metrics3;
          break;
        }
        case 'r1': {
          ds = ridgedata;
          break;
        }
        case 'r2': {
          ds = ridgemapdata;
          break;
        }
        case 't1': {
          ds = this.testData;
          break;
        }
        case 't2': {
          ds = this.testData1K;
          break;
        }
      }

      this.dbdata.setWData('wSLM', {
        data: Object.freeze(ds),
        line: this.lineEnabled,
        colorInterpolateScheme: this.colorInterpolator
      });
    }
  }
};
</script>
