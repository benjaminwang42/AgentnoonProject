<script>
import * as d3 from "d3";
import csvData from "./assets/GigaCorp.csv?raw";

export default {
  name: "App",
  components: {},
  data() {
    return {
      parsed: [],
      roots: [],
      displayHierarchy: true,
    };
  },
  mounted() {
    this.parsed = d3.csvParse(csvData).splice(0, 100);
    // this.parsed = d3.csvParse(csvData);
    this.makeTree(this.parsed);
    this.roots.forEach((root) => {
      this.calculateCosts(root);
    });
    this.drawCharts();
  },
  methods: {
    makeTree(data) {
      const employeeMap = new Map();

      data.forEach((person) => {
        person.name = person.Name;
        delete person.Name;
        person.children = [];
        person.managementCost = 0;
        person.ICCost = 0;
        person.totalCost = 0;
        person.managementCostRatio = 0;

        employeeMap.set(person["Employee Id"], person);
      });

      data.forEach((person) => {
        if (person.Manager && employeeMap.has(person.Manager)) {
          employeeMap.get(person.Manager).children.push(person);
        } else {
          this.roots.push(person);
        }
      });
    },
    calculateCosts(person) {
      let managementCost = 0;
      let ICCost = 0;
      let totalCost = 0;
      for (const child of person.children) {
        this.calculateCosts(child);
        totalCost += child.totalCost;

        if (child.children.length > 0) {
          managementCost += Number(child.Salary);
          managementCost += child.managementCost;
          ICCost += child.ICCost;
        } else {
          ICCost += Number(child.Salary);
        }
      }

      person.managementCost = managementCost;
      person.ICCost = ICCost;
      person.totalCost = totalCost;
      person.managementCostRatio = ICCost
        ? parseFloat((managementCost / ICCost).toFixed(2))
        : -1;
    },
    drawCharts() {
      this.$refs.chartContainer.innerHTML = "";
      this.roots.forEach((rootData) => {
        const svg = this.createTreeChart(rootData);
        this.$refs.chartContainer.appendChild(svg);
      });
    },
    createTreeChart(data) {
      const marginTop = 80;
      const marginRight = 10;
      const marginBottom = 80;
      const marginLeft = 100;

      const root = d3.hierarchy(data);
      const dx = 100;
      const dy = 200;
      const tree = d3.tree().nodeSize([dx, dy]);
      tree(root);

      let minX = Infinity,
        maxX = -Infinity,
        minY = Infinity,
        maxY = -Infinity;

      root.each((d) => {
        if (d.x < minX) minX = d.x;
        if (d.x > maxX) maxX = d.x;
        if (d.y < minY) minY = d.y;
        if (d.y > maxY) maxY = d.y;
      });

      const width = maxY - minY + marginLeft + marginRight;
      const height = maxX - minX + marginTop + marginBottom;

      const diagonal = d3
        .linkHorizontal()
        .x((d) => d.y)
        .y((d) => d.x);

      const svg = d3
        .create("svg")
        .attr("width", width)
        .attr("height", height)
        .attr("viewBox", [minY - marginLeft, minX - marginTop, width, height])
        .attr(
          "style",
          "max-width: none; height: auto; font: 10px sans-serif; user-select: none;"
        );

      const gLink = svg
        .append("g")
        .attr("fill", "none")
        .attr("stroke", "#555")
        .attr("stroke-opacity", 0.4)
        .attr("stroke-width", 1.5);

      const gNode = svg
        .append("g")
        .attr("cursor", "pointer")
        .attr("pointer-events", "all");

      root.x0 = dy / 2;
      root.y0 = 0;
      root.descendants().forEach((d, i) => {
        d.id = i;
        d._children = d.children;
        if (d.depth && d.data.name.length !== 7) d.children = null;
      });

      const update = (event, source) => {
        const duration = event?.altKey ? 2500 : 250;
        const nodes = root.descendants().reverse();
        const links = root.links();

        tree(root);

        let left = root;
        let right = root;
        root.eachBefore((node) => {
          if (node.x < left.x) left = node;
          if (node.x > right.x) right = node;
        });

        const height = right.x - left.x + marginTop + marginBottom;

        const transition = svg
          .transition()
          .duration(duration)
          .attr("height", height)
          .attr("viewBox", [-marginLeft, left.x - marginTop, width, height]);

        const node = gNode.selectAll("g").data(nodes, (d) => d.id);

        const nodeEnter = node
          .enter()
          .append("g")
          .attr("transform", (d) => `translate(${source.y0},${source.x0})`)
          .attr("fill-opacity", 0)
          .attr("stroke-opacity", 0)
          .on("click", (event, d) => {
            d.children = d.children ? null : d._children;
            update(event, d);
          });

        nodeEnter
          .append("foreignObject")
          .attr("x", -80)
          .attr("y", -30)
          .attr("width", 160)
          .attr("height", 70)
          .html(
            (d) => `
          <div xmlns="http://www.w3.org/1999/xhtml" style="
            background: white;
            border: 1px solid #ccc;
            border-radius: 8px;
            padding: 6px 8px;
            font-size: 10px;
            line-height: 1.3;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
            width: 140px;
          ">
            <div><strong>${d.data.name}</strong></div>
            <div>Salary: $${(+d.data.Salary || 0).toLocaleString()}</div>
            <div>IC: $${d.data.ICCost?.toFixed(0) ?? "N/A"}</div>
            <div>Mgmt: $${d.data.managementCost?.toFixed(0) ?? "N/A"}</div>
          </div>
        `
          );

        node
          .merge(nodeEnter)
          .transition(transition)
          .attr("transform", (d) => `translate(${d.y},${d.x})`)
          .attr("fill-opacity", 1)
          .attr("stroke-opacity", 1);

        node
          .exit()
          .transition(transition)
          .remove()
          .attr("transform", (d) => `translate(${source.y},${source.x})`)
          .attr("fill-opacity", 0)
          .attr("stroke-opacity", 0);

        const link = gLink.selectAll("path").data(links, (d) => d.target.id);

        const linkEnter = link
          .enter()
          .append("path")
          .attr("d", (d) => {
            const o = { x: source.x0, y: source.y0 };
            return diagonal({ source: o, target: o });
          });

        link.merge(linkEnter).transition(transition).attr("d", diagonal);

        link
          .exit()
          .transition(transition)
          .remove()
          .attr("d", (d) => {
            const o = { x: source.x, y: source.y };
            return diagonal({ source: o, target: o });
          });

        root.eachBefore((d) => {
          d.x0 = d.x;
          d.y0 = d.y;
        });
      };

      update(null, root);
      return svg.node();
    },
    selectPage(view) {
      if (view == "hierarchy" && this.displayHierarchy == false) {
        this.displayHierarchy = true;
        this.$nextTick(() => {
          this.drawCharts();
        });
      } else if (view == "bonus" && this.displayHierarchy == true) {
        this.displayHierarchy = false;
      }
    },
  },
};
</script>

<template>
  <div class="container">
    <div style="display: flex; justify-content: center; gap: 16px">
      <button @click="selectPage('hierarchy')">Hierarchy</button>
      <button @click="selectPage('bonus')">Bonus</button>
    </div>
    <div v-if="displayHierarchy" ref="chartContainer"></div>
  </div>
</template>

<style scoped></style>
