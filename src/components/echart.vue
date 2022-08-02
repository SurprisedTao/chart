<template>
  <div class="box">
    <div class="chart" ref="chart" v-once></div>
    <el-button @click="clickHandle">{{isBush ? '结束框选' : '框选数据'}}</el-button>
  </div>
</template>

<script>
import * as echarts from "echarts";

function randomData() {
  const now = new Date();
  return {
    name: now.toString(),
    value: [
      [now.getMinutes(), now.getSeconds()].join(":"),
      Math.round(Math.random() * 100),
    ],
  };
}

export default {
  data() {
    return {
      chartData: [randomData()],
      isBush: false,
    };
  },

  methods: {
    setData() {
      const startValue = this.lock
        ? this.lock - 100
        : this.chartData.length - 101;
      const endValue = this.lock || this.chartData.length - 1;

      this.myChart.setOption({
        dataZoom: [
          {
            type: "slider",
            show: true,
            startValue: this.chartData[Math.max(startValue, 0)].value[0],
            endValue: this.chartData[endValue].value[0],
          },
        ],
        series: [
          {
            data: this.chartData,
          },
        ],
      });
    },

    lockHandle() {
      this.lock = this.lock || this.chartData.length - 1;
    },

    clickHandle() {
      this.isBush = !this.isBush;

      if (this.isBush) {
        this.lockHandle();
        this.myChart.dispatchAction({
          type: "takeGlobalCursor",
          key: "brush",
          brushOption: {
            brushType: "lineX",
          },
        });
      } else {
        this.myChart.dispatchAction({
          type: "takeGlobalCursor",
          key: "brush",
          brushOption: {
            brushType: false,
          },
        });
      }
    },
  },

  mounted() {
    this.lock = null;
    this.myChart = echarts.init(this.$refs.chart);
    this.myChart.setOption({
      xAxis: {
        type: "category",
        splitLine: {
          show: false,
        },
      },
      yAxis: {
        type: "value",
        max: 100,
        min: 0,
        splitLine: {
          show: false,
        },
      },
      dataZoom: [
        {
          type: "slider",
          show: true,
          startValue:
            this.chartData[Math.max(this.chartData.length - 101, 0)].value[0],
          endValue: this.chartData[this.chartData.length - 1].value[0],
        },
      ],
      toolbox: { show: false },
      brush: {
        type: "lineX",
        xAxisIndex: "all",
        transformable: false,
        throttleType: "debounce",
        removeOnClick: true,
        brushMode: "multiple",
        brushStyle: {
          color: "rgba(255,36,36,0.2)",
        },
      },
      series: [
        {
          name: "Fake Data",
          type: "line",
          showSymbol: false,
          data: this.chartData,
        },
      ],
    });

    this.myChart.on("datazoom", (e) => {
      if (e.end !== 100) {
        this.lock = this.myChart.getOption().dataZoom[0].endValue;
      } else {
        this.lock = null;
      }
    });

    setInterval(() => {
      this.chartData.push(randomData());
      this.setData();
    }, 1000);
  },
};
</script>

<style lang='less' scoped>
.box {
  display: flex;
  justify-content: center;
  align-items: center;

  .chart {
    width: 800px;
    height: 500px;
  }

  .dropRect {
    position: fixed;
    top: 50px;
    left: 20px;
    width: 0;
    height: 0;
    border: 1px solid #409eff;
    background: rgba(64, 158, 255, 0.2);
  }
}
</style>