<template>
  <div class="box">
    <div class="chart" ref="chart" v-once></div>
  </div>
</template>

<script>
import { Chart } from "@antv/g2";
import random from "random";
// import Dayjs from "dayjs";
export default {
  data() {
    return {
      chartData: [],
    };
  },

  watch: {
    chartData(val) {
      this.$_chart?.data(val);
      this.$_chart?.render();
    },
  },

  methods: {
    addData() {
      this.chartData.push({
        date: Date.now(),
        value: random.int(1, 100),
      });
    },
  },

  mounted() {
    this.$_chart = new Chart({
      container: this.$refs.chart,
      autoFit: true,
      height: 400,
      padding: [16, 32, 64, 32],
    });
    this.$_chart.data(this.chartData);
    this.$_chart.scale({
      date: {
        type: "linear",
        formatter: (date) => {
          return date;
        },
      },
      value: { max: 100, min: 0 },
    });
    this.$_chart.option("scrollbar", {
      type: "horizontal",
    });
    this.$_chart.line().position("date*value");
    this.$_chart.render();

    // this.$timer = setInterval(() => {
    //   this.addData();
    // }, 1000);
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
}
</style>