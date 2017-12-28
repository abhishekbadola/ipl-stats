<template>
  <div>
      <div id="chart-container"></div>
  </div>
</template>
<script>
import Highcharts from "highcharts/highcharts.js";
import { bus } from "../main";

export default {
  name: "Chart",
  created() {
    let $this = this;
    bus.$on("bps", function(bps) {
      // emitted boundaries per season captured here
      $this.boundariesPerSeason = bps;
    });
  },
  mounted() {
    let $this = this,
      interval = setInterval(function() { // To track once all data has loaded
        $this.boundariesPerSeason = JSON.parse(
          window.localStorage.getItem("boundariesPerSeason")
        );
        if ($this.boundariesPerSeason) {
          $this.fillChartsData();
          clearInterval(interval);
        }
      }, 500);
  },
  data() {
    return {
      boundariesPerSeason: null,
      options: {}
    };
  },
  methods: {
    fillChartsData: function() {
      this.boundariesPerSeason = JSON.parse(
        window.localStorage.getItem("boundariesPerSeason")
      );
      let xAxis = [],
        fours = [],
        sixes = [],
        bps = this.boundariesPerSeason;

      for (var season in bps) {
        xAxis.push(season);
        fours.push(bps[season].fours);
        sixes.push(bps[season].sixes);
      }

      this.options = {
        title: {
          text: "Boundaries per season"
        },
        tooltip: {
          shared: true
        },
        credits: {
          enabled: false
        },
        yAxis: {
          title: {
            text: "Boundaries",
            style: {
              fontWeight: "bold"
            }
          }
        },
        xAxis: {
          title: {
            text: "IPL Seasons",
            margin: 20,
            style: {
              fontWeight: "bold"
            }
          },
          categories: xAxis
        },
        series: [
          {
            type: "column",
            name: "Fours",
            data: fours
          },
          {
            type: "line",
            name: "Sixes",
            data: sixes
          }
        ]
      };
      Highcharts.chart("chart-container", this.options);
    }
  }
};
</script>
<style scoped>

</style>
