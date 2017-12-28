<template>
  <div class="home-container">    
    <md-table md-card>
      <md-table-toolbar>
        <h1 class="md-title">{{ 'Season ' + selectedYear + ' Leaderboard' }}</h1>
      </md-table-toolbar>
      <md-table-row>
        <md-table-head>Team Name</md-table-head>
        <md-table-head>Played</md-table-head>
        <md-table-head>Won</md-table-head>
        <md-table-head>Lost</md-table-head>
        <md-table-head>No Results</md-table-head>
        <md-table-head>Form</md-table-head>
      </md-table-row>

      <md-table-row v-for="(value, index) in pointsTable" :key="value.Name + index">
        <md-table-cell>{{value.Name}}</md-table-cell>
        <md-table-cell>{{value.Played}}</md-table-cell>
        <md-table-cell>{{value.Won}}</md-table-cell>
        <md-table-cell>{{value.Lost}}</md-table-cell>
        <md-table-cell>{{value.NoResult}}</md-table-cell>
        <md-table-cell>
          <md-chip v-for="(chip, index) in value.Form" :key="index" :class="{'md-accent': (chip == 'L'), 'md-primary': (chip == 'W')}">{{ chip }}</md-chip>
        </md-table-cell>
      </md-table-row>
    </md-table>
  </div>
</template>

<script>
const storage = window.localStorage;
import { bus } from "../main";
import Papa from "papaparse/papaparse.min.js";

export default {
  name: "Home",
  data() {
    return {
      ballsData: [],
      matchesData: [],
      matchesByYear: [],
      selectedYear: (sessionStorage.getItem('season') ? sessionStorage.getItem('season') : "2016"),
      pointsTableDictionary: {}, // For data dictionary
      pointsTable: [],
      matchesPerSeason: {},
      boundariesPerSeason: {}
    };
  },
  created() { // Getting data once home instance created
    this.getData("match");
    this.getData("ball");
    let t = this;
    bus.$on("season", function(season) {
      sessionStorage.setItem('season', season);
      t.selectedYear = season;
      t.getData("match");
    });
  },
  methods: {
    getData: function(key) {
      let chunkCount = 0,
        ballByBallUrl = "./static/Ball_by_ball.csv",
        matchesUrl = "./static/Match.csv";

      switch (key) {
        case "ball":
          let bd = storage.getItem("ballsData");
          let d = this.ballsData;
          if (!bd) {
            this.csvToJson(ballByBallUrl, "ball"); // getting ball by ball data from csv file
          }
          break;
        case "match":
          let year = this.selectedYear,
            ls = storage.getItem("pointsTableDictionary-" + year),
            csvFlag = 0; // To check if matches data is loading or not

          if (!this.matchesData.length) {
            let smd = localStorage.getItem("matchesData");
            if (smd) {
              this.matchesData = JSON.parse(smd);
            } else {
              csvFlag = 1;
              // getting matches data from csv file
              this.csvToJson(matchesUrl, "match");
            }
          }

          if (!ls) {
            // Filtering by season if season is changed
            if (this.matchesData.length && !csvFlag) {
              this.filterMatchesDataByYear(this.selectedYear);
            }
          } else {
            // If current slection data is available in Local Storage
            this.pointsTableDictionary[year] = JSON.parse(ls);
            this.fillPointsTable();
          }
          break;
      }
    },
    csvToJson: function(remoteUrl, dataKey) {
      let $this = this;
      Papa.parse(remoteUrl, {
        download: true,
        header: true,
        //fastMode: true,
        error: $this.papaErrorCallback,
        chunk: function(results, parser) {
          $this.papaSuccessCallback(results, parser, dataKey);
        }
      });
    },
    papaErrorCallback: function(err, file) {
      console.log("ERROR:", err, file);
    },
    papaSuccessCallback: function(results, parser, dataKey) {
      let data = results.data;

      switch (dataKey) {
        case "ball":
          Array.prototype.push.apply(this.ballsData, data);
          if (parser.streamer._finished) {
            this.ballsData = this.ballsData; // Refresing the data
            this.getBoundariesPerSeason();
          }
          break;
        case "match":
          Array.prototype.push.apply(this.matchesData, data);
          if (parser.streamer._finished) {
            this.matchesData = this.matchesData; // Refreshing the data
            storage.setItem("matchesData", JSON.stringify(this.matchesData));
            this.getMatchesPerSeason();
            this.filterMatchesDataByYear(this.selectedYear);
          }
          break;
      }
    },
    getBoundariesPerSeason: function() {
      let bps = storage.getItem("boundariesPerSeason");
      if (bps) {
        this.boundariesPerSeason = JSON.parse(bps);
      } else {
        let mps = this.matchesPerSeason;
        let bps = {}; // boundaries per season

        let $this = this;
        this.ballsData.forEach(function(item) {
          let bs = item.Batsman_Scored;
          if (bs == 6 || bs == 4) {
            // Filtering only boundaries
            let season = new Date(
              $this.matchesData.find(function(match) {
                return match.Match_Id == this;
              }, item.Match_Id).Match_Date
            ).getFullYear();

            if (!bps[season]) {
              bps[season] = {
                sixes: 0,
                fours: 0
              };
            }

            if (bs == 6) bps[season].sixes += 1;
            if (bs == 4) bps[season].fours += 1;
          }
        });

        this.boundariesPerSeason = bps;
        localStorage.setItem(
          "boundariesPerSeason",
          JSON.stringify(this.boundariesPerSeason)
        );
      }

      bus.$emit('bps', this.boundariesPerSeason);
    },
    getMatchesPerSeason: function() {
      let md = this.matchesData,
        mps = this.matchesPerSeason;

      md.forEach(function(item) {
        // creating dictionary of matches per season
        let fullYear = new Date(item.Match_Date).getFullYear(),
          sd = mps[fullYear]; // Season dictionary

        if (!sd) {
          sd = [];
        }
        sd.push(item.Match_Id);

        if (fullYear) mps[fullYear] = sd;
      });
      this.matchesPerSeason = mps;
      console.log(mps);
    },
    fillPointsTable: function() {
      let md = this.pointsTableDictionary[this.selectedYear];
      this.pointsTable = [];
      for (let prop in md) {
        this.pointsTable.push(md[prop]);
      }
      this.pointsTable.sort(function(a, b) {
        return b.Won - a.Won;
      });
    },
    filterMatchesDataByYear: function(year) {
      // Filtering matches data by year selection
      let fullData = this.matchesData,
        ls = storage.getItem("pointsTableDictionary-" + year),
        pointsDict = this.pointsTableDictionary[year];
      this.selectedYear = year;

      let t = this;
      let filteredMatchesByYear = fullData.filter(function(item) {
        let fullYear = new Date(item.Match_Date).getFullYear();
        return fullYear == year;
      });

      if (filteredMatchesByYear) {
        if (!pointsDict) t.pointsTableDictionary[year] = pointsDict = {};

        filteredMatchesByYear.forEach(element => {
          let ftId = element.Team_Name_Id, // First team Id
            oId = element.Opponent_Team_Id, // Opponent team Id
            dictT = pointsDict[ftId], // First team dictionary
            dictO = pointsDict[oId], // Opponent team dictionary
            matchWinner = element.Match_Winner_Id,
            isResult = parseInt(element.IS_Result);

          for (let i = 0; i < 2; i++) {
            let dict = i ? dictT : dictO, // Setting variable for each team one by one
              tId = i ? ftId : oId;

            if (dict) {
              dict["Played"] += 1;

              dict["Won"] += isResult && matchWinner == tId ? 1 : 0;

              dict["Lost"] += isResult && matchWinner != tId ? 1 : 0;

              dict["NoResult"] += !isResult ? 1 : 0;

              let dfLen = dict["Form"].length;
              if (dfLen == 5) {
                dict["Form"].splice(0, 1);
              }
            } else {
              dict = {
                Name: "Team-" + tId,
                Form: []
              };

              dict["Played"] = 1;

              dict["Won"] = isResult && matchWinner == tId ? 1 : 0;

              dict["Lost"] = isResult && matchWinner != tId ? 1 : 0;

              dict["NoResult"] = !isResult ? 1 : 0;
            }

            if (!isResult) {
              dict["Form"].push("NR");
            } else if (isResult && matchWinner == tId) {
              dict["Form"].push("W");
            } else {
              dict["Form"].push("L");
            }

            t.pointsTableDictionary[year][tId] = dict;
          }
        });

        storage.setItem(
          // Storing points table as season wise to reuse
          "pointsTableDictionary-" + year,
          JSON.stringify(this.pointsTableDictionary[year])
        );

        this.fillPointsTable();
      }
    }
  }
};
</script>
<style>
.md-chip {
  width: 25px;
  height: 25px;
  padding: 0px 0px;
  text-align: center;
}
.md-chip.md-theme-default.md-primary {
  background-color: #6dc06d;
}
.md-table-cell:last-child .md-table-cell-container {
  min-width: 220px;
}
</style>
