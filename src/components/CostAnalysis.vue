<template>
  <div class="pl-16">
    <div class="ml-60 mb-6">
      <select
        v-model="currentCategory"
        class="border border-gray-300 rounded p-2 mr-4 text-sm mb-2"
      >
        <option value="entity">Entity View</option>
        <option value="level">Level View</option>
        <option value="department">Department View</option>
      </select>
      <select
        v-model="sumType"
        class="border border-gray-300 rounded p-2 mr-4 text-sm mb-2"
      >
        <option value="actuals">Actuals</option>
        <option value="percentages">Percentages</option>
      </select>
      <select
        v-model="aggregateType"
        class="border border-gray-300 rounded p-2 text-sm"
      >
        <option value="totals">Totals</option>
        <option value="averages">Averages</option>
      </select>
      <div></div>
    </div>
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
  watch: {
    currentCategory() {
      this.updateCategory();
    },
    sumType() {
      this.updateCategory();
    },
    aggregateType() {
      this.updateCategory();
    },
  },
  data() {
    return {
      sumType: "actuals",
      aggregateType: "totals",
      currentCategory: "entity",
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
        title: {
          text: "Headcount Cost Distribution (Partitioned into Salary and Bonus)",
          left: "center",
          top: 10,
          textStyle: {
            fontSize: 18,
            fontWeight: "bold",
          },
        },
        grid: { left: 240 },
        tooltip: {
          trigger: "axis",
          axisPointer: { type: "shadow" },
        },
        legend: { top: 40, data: ["Salary", "Bonus"] },
        xAxis: {
          type: "value",
          name: "Total Cost",
          nameLocation: "middle",
          nameGap: 40,
          nameTextStyle: {
            fontSize: 18,
            fontWeight: "bold",
          },
        },
        yAxis: {
          type: "category",
          data: [],
          name: "Total Cost in Dollars",
          nameLocation: "middle",
          nameGap: 220,
          nameTextStyle: {
            fontSize: 18,
            fontWeight: "bold",
          },
        },
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

    this.updateCategory();
  },
  methods: {
    updateCategory() {
      let newCategories = [];
      let newSalaryData = [];
      let newBonusData = [];
      let categoryName = "";
      let yAxisGap = 170;

      if (this.currentCategory === "entity") {
        newCategories = this.entityCategories;
        newSalaryData = this.entitySalaryData;
        newBonusData = this.entityBonusData;
        categoryName = "Entity";
        yAxisGap = 170;
      } else if (this.currentCategory === "level") {
        newCategories = this.levelCategories;
        newSalaryData = this.levelSalaryData;
        newBonusData = this.levelBonusData;
        categoryName = "level";
        yAxisGap = 50;
      } else if (this.currentCategory === "department") {
        newCategories = this.departmentCategories;
        newSalaryData = this.departmentSalaryData;
        newBonusData = this.departmentBonusData;
        categoryName = "Department";
        yAxisGap = 220;
      }

      if (this.aggregateType === "averages") {
        newSalaryData = newCategories.map((category, index) => {
          const members = this.data.filter(
            (person) => person[categoryName] === category
          );
          return newSalaryData[index] / members.length;
        });

        newBonusData = newCategories.map((category, index) => {
          const members = this.data.filter(
            (person) => person[categoryName] === category
          );
          return newBonusData[index] / members.length;
        });
      }

      if (this.sumType === "percentages") {
        const total =
          newSalaryData.reduce((a, b) => a + b, 0) +
          newBonusData.reduce((a, b) => a + b, 0);
        newSalaryData = newSalaryData.map((val) => (val / total) * 100);
        newBonusData = newBonusData.map((val) => (val / total) * 100);
      }

      const container = this.$refs.costAnalysisChart;
      const existing = echarts.getInstanceByDom(container);
      if (existing) {
        echarts.dispose(container);
      }

      const chart = echarts.init(this.$refs.costAnalysisChart);

      newSalaryData = newSalaryData.map((num) => Number(num.toFixed(2)));
      newBonusData = newBonusData.map((num) => Number(num.toFixed(2)));
      const option = {
        title: {
          text: "Headcount Cost Distribution (Partitioned into Salary and Bonus)",
          left: "center",
          top: 10,
          textStyle: {
            fontSize: 18,
            fontWeight: "bold",
          },
        },
        grid: { left: 240 },
        tooltip: {
          trigger: "axis",
          axisPointer: { type: "shadow" },
        },
        legend: { top: 40, data: ["Salary", "Bonus"] },
        xAxis: {
          type: "value",
          name: "Total Cost",
          nameLocation: "middle",
          nameGap: 40,
          nameTextStyle: {
            fontSize: 18,
            fontWeight: "bold",
          },
        },
        yAxis: {
          type: "category",
          data: newCategories,
          name: categoryName,
          nameLocation: "middle",
          nameGap: yAxisGap,
          nameTextStyle: {
            fontSize: 18,
            fontWeight: "bold",
          },
        },
        series: [
          {
            name: "Salary",
            type: "bar",
            stack: "total",
            data: newSalaryData,
          },
          {
            name: "Bonus",
            type: "bar",
            stack: "total",
            data: newBonusData,
          },
        ],
      };
      chart.setOption(option);
      // this.chartOption.yAxis.data = newCategories;
      // this.chartOption.series[0].data = newSalaryData;
      // this.chartOption.series[1].data = newBonusData;
      // this.chartOption.yAxis.name = categoryName;
      // this.chartOption.yAxis.nameGap = yAxisGap;
      // this.chart.setOption(this.chartOption);
    },
  },
};
</script>

<style scoped>
.echarts-tooltip {
  z-index: 9999 !important;
  pointer-events: auto !important;
  background: rgba(255, 255, 255, 0.9) !important;
  color: black !important;
}
</style>
