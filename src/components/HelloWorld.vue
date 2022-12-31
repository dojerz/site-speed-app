<script>
//import { ref } from "vue";
import * as d3 from "d3";

export default {
  name: "HelloWorld",
  data() {
    return {
      isLoading: true,
      localDataUrl: "./data/global-temperatures-datahub.csv",
      data: [], // processed data
    };
  },
  updated: {
    chart() {
      const svg = d3
        .create("svg")
        .attr("viewBox", [
          0,
          0,
          width,
          innerHeight + margin.top + margin.bottom,
        ])
        .attr("font-family", "sans-serif")
        .attr("font-size", 10);

      svg.append("g").call(xAxis);

      svg.append("g").call(yAxis);

      svg
        .append("g")
        .selectAll("g")
        .data(data.values)
        .join("g")
        .attr("transform", (d, i) => `translate(0,${y(data.names[i])})`)
        .selectAll("rect")
        .data((d) => d)
        .join("rect")
        .attr("x", (d, i) => x(data.years[i]) + 1)
        .attr("width", (d, i) => x(data.years[i] + 1) - x(data.years[i]) - 1)
        .attr("height", y.bandwidth() - 1)
        .attr("fill", (d) => (isNaN(d) ? "#eee" : d === 0 ? "#fff" : color(d)))
        .append("title")
        .text((d, i) => `${format(d)} per 100,000 people in ${data.years[i]}`);

      return svg.node();
    },
  },
  mounted() {
    this.loadData();
    this.createChart();
  },
  async created() {},
  methods: {
    async loadData() {
      const names = [
        "Alaska",
        "Ala.",
        "Ark.",
        "Ariz.",
        "Calif.",
        "Colo.",
        "Conn.",
        "D.C.",
        "Del.",
        "Fla.",
        "Ga.",
        "Hawaii",
        "Iowa",
        "Idaho",
        "Ill.",
        "Ind.",
        "Kan.",
        "Ky.",
        "La.",
        "Mass.",
        "Md.",
        "Maine",
        "Mich.",
        "Minn.",
        "Mo.",
        "Miss.",
        "Mont.",
        "N.C.",
        "N.D.",
        "Neb.",
        "N.H.",
        "N.J.",
        "N.M",
        "Nev.",
        "N.Y.",
        "Ohio",
        "Okla.",
        "Ore.",
        "Pa.",
        "R.I.",
        "S.C.",
        "S.D.",
        "Tenn.",
        "Texas",
        "Utah",
        "Va.",
        "Vt.",
        "Wash.",
        "Wis.",
        "W.Va.",
        "Wyo.",
      ];
      let data = await d3.json(new URL("vaccines.json", import.meta.url));
      const values = [];
      const year0 = d3.min(data[0].data.values.data, (d) => d[0]);
      const year1 = d3.max(data[0].data.values.data, (d) => d[0]);
      const years = d3.range(year0, year1 + 1);
      for (const [year, i, value] of data[0].data.values.data) {
        if (value == null) continue;
        (values[i] || (values[i] = []))[year - year0] = value;
      }

      this.chdata.values = values;
      this.chdata.names = names;
      this.chdata.years = years;
      this.chdata.year = data[0].data.chart_options.vaccine_year;
      console.log(data);
    },
    createChart() {
      let x = d3
        .scaleLinear()
        .domain([d3.min(this.chdata.years), d3.max(this.chdata.years) + 1])
        .rangeRound([margin.left, width - margin.right]);
      let y = d3
        .scaleBand()
        .domain(chdata.names)
        .rangeRound([margin.top, margin.top + innerHeight]);
      let xAxis = (g) =>
        g
          .call((g) =>
            g
              .append("g")
              .attr("transform", `translate(0,${margin.top})`)
              .call(d3.axisTop(x).ticks(null, "d"))
              .call((g) => g.select(".domain").remove())
          )
          .call((g) =>
            g
              .append("g")
              .attr("transform", `translate(0,${innerHeight + margin.top + 4})`)
              .call(
                d3
                  .axisBottom(x)
                  .tickValues([data.year])
                  .tickFormat((x) => x)
                  .tickSize(-innerHeight - 10)
              )
              .call((g) =>
                g
                  .select(".tick text")
                  .clone()
                  .attr("dy", "2em")
                  .style("font-weight", "bold")
                  .text("Measles vaccine introduced")
              )
              .call((g) => g.select(".domain").remove())
          );
      let yAxis = (g) =>
        g
          .attr("transform", `translate(${margin.left},0)`)
          .call(d3.axisLeft(y).tickSize(0))
          .call((g) => g.select(".domain").remove());
    },
  },
};
</script>

<template>
  <div v-if="isLoading">Loading data...</div>
  <div v-if="chdata">
    <svg></svg>
  </div>

  <div class="card">
  </div>
</template>

<style scoped>
.read-the-docs {
  color: #888;
}
</style>
