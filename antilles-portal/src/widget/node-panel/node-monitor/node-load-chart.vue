<style scoped>
</style>
<template>
<div ref="container" class="node-monitor-chart-style"></div>
</template>
<script>
import ECharts from 'echarts'
import MonitorDataService from '../../../service/monitor-data'
import Formatter from '../../../common/format'

export default {
  data() {
    return {
      innerChart: null,
      innerNodeId: 0
    }
  },
  props: [
    'nodeId',
    'interval'
  ],
  watch: {
    nodeId(val, oldVal) {
      this.innerNodeId = val;
      this.clear();
      this.refresh();
    }
  },
  mounted() {
    this.refresh();
  },
  mounted() {
    this.innerNodeId = this.nodeId;
    this.$nextTick(() => {
      this.innerChart = ECharts.init(this.$refs.container);
      window.removeEventListener('resize', this.resizeChart);
      window.addEventListener('resize', this.resizeChart);
      this.resizeChart();
      this.init();
      this.refresh();
      gApp.$watch('isCollapse', (newValue, oldValue) => {
        setTimeout(() => {
          this.resizeChart();
        },300)
      })
    });
  },
  beforeDestroy() {
    this.innerNodeId = 0;
    window.removeEventListener('resize', this.resizeChart);
  },
  methods: {
    resizeChart() {
      this.innerChart.resize();
    },
    init() {
      var option = {
        title: {
          text: this.$t('NodeDetail.LOAD.Unit'),
          textStyle: {
            fontSize: 14,
            color: "#333333"
          },
          padding: [10, 5]
        },
        tooltip: {
          trigger: 'axis'
        },
        grid: {
          top: 40,
          bottom: 30,
          left: 50,
          right: 30
        },
        xAxis: {
          type: 'time',
          splitLine: {
            show: false
          },
          axisLine: {
            lineStyle: {
              color: '#AAAAAA'
            }
          },
          axisLabel: {
            color: '#999999',
            formatter: function(value){
              return Formatter.formatDateTime(new Date(value), 'hh:mm');
            }
          },
          axisTick: {
            show: false
          }
        },
        yAxis: {
          type: 'value',
          name: '',
          axisLine: {
            lineStyle: {
              color: '#AAAAAA'
            },
            show: false
          },
          axisLabel: {
            formatter: '{value}',
            color: '#999999'
          },
          axisTick: {
            show: false
          }
        },
        color: ['#62b5f9'],
        series: [{
          name: this.$t('NodeDetail.LOAD.Unit'),
          type: 'line',
          yAxisIndex: 0,
          showSymbol: false,
          smooth: true,
          itemStyle : {
            normal : {
              lineStyle:{
                shadowBlur: 6,
                shadowColor: '#f1f8fe',
                color:'#62b5f9',
                width: 1
              }
            }
          },
          areaStyle: {
            normal: {
              color: new ECharts.graphic.LinearGradient(0, 0, 0, 1, [{
                  offset: 0,
                  color: '#eff7fe'
              }])
            }
          },
          data: []
        }]
      };
      this.innerChart.setOption(option);
    },
    chartDataMapping(timeSeriesItem) {
      return {
        value: [
          timeSeriesItem.time,
          timeSeriesItem.values[0]
        ]
      };
    },
    clear() {
      this.innerChart.setOption({
        series: [{
          data: []
        }]
      });
    },
    refresh() {
      if (this.innerNodeId > 0) {
        MonitorDataService.getNodeDataByHour(this.nodeId, 'load', 1).then((res) => {
          if(this.innerNodeId == res.id){
            var loadData = [];
            res.data.forEach((item) => {
              loadData.push(this.chartDataMapping(item));
            });
            this.innerChart.setOption({
              series: [{
                data: loadData
              }]
            });
            let self = this;
            setTimeout(self.refresh, this.interval);
          }
        }, (res)=> {
          this.$message.error(res);
        })
      }
    }
  }
}
</script>
