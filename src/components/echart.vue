<template>
  <div class="box">
    <div>
      <el-button @click="clickHandle">{{isBush ? '结束框选' : '框选数据'}}</el-button>
      <div class="chart" ref="chart" v-once></div>
    </div>

    <ul class="menu" :style="menuStyle" tabindex="0" @blur="hideContextmenu" ref="contextmenu">
      <li>同步至云端</li>
      <li>添加场景标签</li>
    </ul>
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
      menuStyle: {},
    };
  },

  methods: {
    setData() {
      const endValue = this.lock || this.chartData.length - 1;
      const startValue = Math.max(endValue - this.scrollLength, 0);
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

    hideContextmenu() {
      this.menuStyle = {};
    },

    caclScroll() {},
  },

  mounted() {
    this.lock = null;
    this.scrollLength = 30;

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
      this.scrollLength =
        this.myChart.getOption().dataZoom[0].endValue -
        this.myChart.getOption().dataZoom[0].startValue;
      if (e.end !== 100) {
        this.lock = this.myChart.getOption().dataZoom[0].endValue;
      } else {
        this.lock = null;
      }
    });

    this.myChart.getZr().on("contextmenu", (e) => {
      e.stop();
      if (e.topTarget && e.topTarget.type === "rect") {
        // 展示自定义菜单
        this.menuStyle = {
          display: "block",
          left: e.event.clientX + "px",
          top: e.event.clientY + "px",
        };

        this.$nextTick(() => {
          this.$refs.contextmenu.focus();
        });
      }
    });

    this.myChart.getZr().on("dblclick", (e) => {
      if (e.topTarget && e.topTarget.type === "rect") {
        this.myChart.dispatchAction({
          type: "brush",
          areas: [],
        });
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

  .menu {
    display: none;
    position: fixed;
    left: 0;
    top: 0;
    list-style: none;
    padding: 4px 0;
    margin: 5px 0;
    background-color: #fff;
    border: 1px solid #ebeef5;
    border-radius: 4px;
    box-shadow: 0 2px 12px 0 rgb(0 0 0 / 10%);

    li {
      list-style: none;
      line-height: 36px;
      padding: 0 20px;
      margin: 0;
      font-size: 14px;
      color: #606266;
      cursor: pointer;
      outline: none;

      &:hover {
        background-color: #ecf5ff;
        color: #66b1ff;
      }
    }
  }
}
</style>