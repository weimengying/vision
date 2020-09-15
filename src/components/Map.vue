<template>
  <div class="com-container">
    <div class="com-chart" ref="map_ref"></div>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  data() {
    return {
      chartInstance: null,
      allData: null
    }
  },
  mounted () {
    this.initChart()
    this.getData()
    window.addEventListener('resize', this.screenAdapter)
    // 在页面加载完成的时候, 主动进行屏幕的适配
    this.screenAdapter()
  },
  destroyed () {
    window.removeEventListener('resize', this.screenAdapter)
  },
  methods: {
    async initChart () {
      this.chartInstance = this.$echarts.init(this.$refs.map_ref, 'chalk')
      // 获取中国地图的矢量数据
      // http://localhost:8999/static/map/china.json
      // 由于 我们现在获取的地图矢量数据并不是位于KOA2的后台，所有咱们不能使用this.$http
      const ret = await axios.get('http://localhost:8999/static/map/china.json')
      this.$echarts.registerMap('china', ret.data)
      const initOption = {
        geo: {
          type: 'map',
          map: 'china'
        }
      }
      this.chartInstance.setOption(initOption)
    },
    async getData () {
      // 获取服务器的数据，对this.allData进行赋值之后，调用updataChart方法更新图表
      const { data: ret } = await this.$http.get('map')
      this.allData = ret
      console.log(this.allData)
      this.updateChart()
    },
    updateChart () {
      // 处理图表需要的数据
      // 图例的数据
      const legendArr = this.allData.map(item => {
        return item.name
      })
      const seriesArr = this.allData.map(item => {
        // return的这个对象就代表的是一个类别下的所有散点数据
        return {
          type: 'effectScatter',
          name: item.name,
          data: item.children
        }
      })
      const dataOption = {
        legend: {
          data: legendArr
        },
        series: seriesArr
      }
      this.chartInstance.setOption(dataOption)
    },
    screenAdapter () {
      const adapterOption = {}
      this.chartInstance.setOption(adapterOption)
      this.chartInstance.resize()
    }
  }
}
</script>
<style>
</style>
