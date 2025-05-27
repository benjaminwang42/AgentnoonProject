<template>
  <div class="pl-16">
    <button class="m-0" @click="switchToEntityData">
      Switch to Entity View
    </button>
    <button class="m-0" @click="switchToLevelData">Switch to Level View</button>
    <button class="m-0" @click="switchToDeptData">
      Switch to Department View
    </button>
    <div ref="costAnalysisChart" style="width: 100%; height: 700px"></div>
  </div>
</template>

<script>
import * as echarts from "echarts";

export default {
  name: "StackedBarChart",
  props: {
    data: {
      type: Array,
      required: true,
    },
  },
  data() {
    return {
      entityCategories: [],
      entitySalaryData: [],
      entityBonusData: [],
      levelCategories: [],
      levelSalaryData: [],
      levelBonusData: [],
      departmentCategories: [],
      departmentSalaryData: [],
      departmentBonusData: [],
      chart: null,
      chartOption: {
        grid: { left: 200 },
        tooltip: {
          trigger: "axis",
          axisPointer: { type: "shadow" },
        },
        legend: { data: ["Salary", "Bonus"] },
        xAxis: { type: "value" },
        yAxis: { type: "category", data: [] },
        series: [
          {
            name: "Salary",
            type: "bar",
            stack: "total",
            data: [],
          },
          {
            name: "Bonus",
            type: "bar",
            stack: "total",
            data: [],
          },
        ],
      },
    };
  },
  mounted() {
    this.entityCategories = [
      ...new Set(this.data.map((person) => person.Entity)),
    ];

    this.entitySalaryData = this.entityCategories.map((category) =>
      this.data
        .filter((item) => item["Entity"] === category)
        .reduce((sum, item) => sum + Number(item.Salary), 0)
    );

    this.entityBonusData = this.entityCategories.map((category) =>
      this.data
        .filter((item) => item["Entity"] === category)
        .reduce((sum, item) => sum + Number(item.Bonus), 0)
    );

    this.levelCategories = [
      ...new Set(this.data.map((person) => person.level)),
    ];

    this.levelSalaryData = this.levelCategories.map((category) =>
      this.data
        .filter((item) => item["level"] === category)
        .reduce((sum, item) => sum + Number(item.Salary), 0)
    );

    this.levelBonusData = this.levelCategories.map((category) =>
      this.data
        .filter((item) => item["level"] === category)
        .reduce((sum, item) => sum + Number(item.Bonus), 0)
    );

    this.departmentCategories = [
      ...new Set(this.data.map((person) => person.Department)),
    ];

    this.departmentSalaryData = this.departmentCategories.map((category) =>
      this.data
        .filter((item) => item["Department"] === category)
        .reduce((sum, item) => sum + Number(item.Salary), 0)
    );

    this.departmentBonusData = this.departmentCategories.map((category) =>
      this.data
        .filter((item) => item["Department"] === category)
        .reduce((sum, item) => sum + Number(item.Bonus), 0)
    );
    this.chart = echarts.init(this.$refs.costAnalysisChart);
    console.log(
      "BEN VARS",
      this.entityCategories,
      this.entitySalaryData,
      this.entityBonusData
    );
    console.log(
      "LEVEL VARS",
      this.levelCategories,
      this.levelSalaryData,
      this.levelBonusData
    );
    console.log(
      "DEPT VARS",
      this.departmentCategories,
      this.departmentSalaryData,
      this.departmentBonusData
    );
    this.applyDataToChart();
  },
  methods: {
    applyDataToChart() {
      this.chartOption.yAxis.data = this.entityCategories;
      this.chartOption.series[0].data = this.entitySalaryData;
      this.chartOption.series[1].data = this.entityBonusData;
      this.chart.setOption(this.chartOption);
    },
    switchToEntityData() {
      this.chartOption.yAxis.data = this.entityCategories;
      this.chartOption.series[0].data = this.entitySalaryData;
      this.chartOption.series[1].data = this.entityBonusData;
      this.chart.setOption(this.chartOption);
    },
    switchToLevelData() {
      this.chartOption.yAxis.data = this.levelCategories;
      this.chartOption.series[0].data = this.levelSalaryData;
      this.chartOption.series[1].data = this.levelBonusData;
      this.chart.setOption(this.chartOption);
    },
    switchToDeptData() {
      this.chartOption.yAxis.data = this.departmentCategories;
      this.chartOption.series[0].data = this.departmentSalaryData;
      this.chartOption.series[1].data = this.departmentBonusData;
      this.chart.setOption(this.chartOption);
    },
  },
};
</script>

<style scoped></style>
