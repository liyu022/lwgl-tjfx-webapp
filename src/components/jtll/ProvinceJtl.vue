<template>
  <div id='chart' :style="'height:'+ height +'px;'">
  </div>
</template>

<script>
// import logger from '../../services/logger'
export default {
  name: '',
  data () {
    return {
      title: '全省高速公路交通量分析',
      chart: null,
      height: 100
    }
  },
  props: {
    params: {
      carType: null,
      interval: null,
      dateYear: '',
      dateMonth: '',
      direction: null,
      fullHeight: 0
    }
  },
  computed: {
    date: function () {
      return this.params.interval === 'y' ? this.params.dateYear : this.params.dateMonth
    }
  },
  created () {
    this.setSize()
  },
  mounted () {
    this.log.logging('JTYJ-APP-GLYX-TJFX', '交通流量', '全省高速公路交通量统计查询')
    this.createChart()
    this.loadData()
  },
  watch: {
    'params.fullHeight' (val) {
      this.setSize()
    },
    'params.carType' (val) {
      this.loadData()
    },
    'params.dateYear' (val) {
      this.loadData()
    },
    'params.dateMonth' (val) {
      this.loadData()
    }
  },
  methods: {
    createChart () {
      this.chart = echarts.init(document.getElementById('chart'))
    },
    loadData () {
      // debugger
      var that = this
      this.$http.getData(config.baseUrl + config.service.jtll.provinceJtl.chartData, {
        date: this.date,
        carType: this.params.carType,
        interval: this.params.interval
      }, {}, function (data, msg) {
        console.log(data)
        // 万辆单位处理
        let hcjtl = []
        let kcjtl = []
        let date = []
        if (data !== null) {
          data.hcjtl.forEach(function (item, index) {
            hcjtl.push((item / 10000).toFixed(2))
          })
          data.kcjtl.forEach(function (item, index) {
            kcjtl.push((item / 10000).toFixed(2))
          })
          date = data.date
        }
        var option = {
          tooltip: {
            trigger: 'axis',
            axisPointer: {
              type: 'line'
            }
          },
          legend: {
            data: ['客车', '货车', '环比']
          },
          toolbox: {
            show: true,
            feature: {
              dataView: {show: true},
              saveAsImage: {
                show: true,
                excludeComponents: ['toolbox'],
                pixelRatio: 2
              }
            }
          },
          grid: {
            left: '3%',
            right: '3%',
            bottom: '3%',
            containLabel: true
          },
          // x轴显示
          xAxis: {
            data: date,
            splitLine: {
              show: false
            },
            axisLabel: {
              formatter: function (value) {
                return value + (that.params.interval === 'y' ? '月' : '日')
              }
            }
          },
          // y轴显示
          yAxis: [
            {
              type: 'value',
              name: '交通量',
              splitLine: {
                show: false
              },
              axisLabel: {
                formatter: '{value} 万辆'
              }
            },
            {
              type: 'value',
              splitLine: {
                show: false
              },
              axisLabel: {
                formatter: '{value}%'
              }
            }
          ],
          series: [
            {
              name: '客车',
              type: 'bar',
              stack: '业务',
              data: kcjtl
            },
            {
              name: '货车',
              type: 'bar',
              stack: '业务',
              data: hcjtl
            },
            {
              name: '环比',
              type: 'line',
              yAxisIndex: 1,
              data: data.ringrate
            }
          ]
        }
        that.chart.setOption(option)
      })
    },
    /**
     * 布局计算
     */
    setSize () {
      const clientHeight = this.params.fullHeight === 0 ? document.documentElement.clientHeight : this.params.fullHeight
      this.height = clientHeight - 108
      console.log('this.height:' + clientHeight)
    }
  }
}
</script>

<style scoped>

</style>
