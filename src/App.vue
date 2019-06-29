<template>
  <v-app>
    <v-toolbar app>
      <v-toolbar-title class="headline text-uppercase">
        <span>Curseforge</span>
        <span class="font-weight-light">Analyzer</span>
      </v-toolbar-title>
      <v-spacer></v-spacer>
    </v-toolbar>

    <v-content>
      <v-flex xs12 class="text-xs-center text-sm-center text-md-center text-lg-center mt-4">
        <v-text-field
          label="Select CSV"
          @click="pickFile"
          v-model="fileName"
          prepend-icon="attach_file"
        ></v-text-field>
        <input
          type="file"
          style="display: none"
          ref="image"
          accept="text/csv"
          @change="onFilePicked"
        >
      </v-flex>
      <v-layout row>
        <v-flex xs3>
          <v-checkbox v-model="selections" label="Daily Downloads" value="daily_downloads"></v-checkbox>
        </v-flex>
        <v-flex xs3>
          <v-checkbox
            v-model="selections"
            label="Daily Curse Downloads"
            value="daily_curse_downloads"
          ></v-checkbox>
        </v-flex>
        <v-flex xs3>
          <v-checkbox
            v-model="selections"
            label="Daily Twitch Downloads"
            value="daily_twitch_downloads"
          ></v-checkbox>
        </v-flex>
        <v-flex xs3>
          <v-checkbox v-model="selections" label="Points" value="points"></v-checkbox>
        </v-flex>
      </v-layout>
      <GChart class="chart" type="LineChart" :data="chartData" :options="chartOptions" fill-height/>
    </v-content>
  </v-app>
</template>

<script>
import moment from "moment";
import { watch } from "fs";

let csvData = [];

export default {
  name: "App",
  components: {},
  data() {
    return {
      chartData: [
        [
          "Date",
          "Daily Downloads",
          "Daily Twitch Downloads",
          "Daily Curse Downloads",
          "Points"
        ],
        ["", 1, 1, 1, 1]
      ],
      chartOptions: {
        curveType: "function",
        vAxis: { viewWindow: { min: 0 } }
      },
      fileName: "",
      selections: [
        "daily_downloads",
        "daily_curse_downloads",
        "daily_twitch_downloads",
        "points"
      ]
    };
  },
  methods: {
    pickFile() {
      this.$refs.image.click();
    },
    onFilePicked(e) {
      const files = e.target.files;
      if (files[0] === undefined) {
        this.fileName = "";
        return;
      }
      this.fileName = files[0].name;
      const reader = new FileReader();
      reader.onload = event => {
        csvData = this.parseCSV(event.target.result);
        this.updateChart();
      };
      reader.readAsText(files[0]);
    },
    updateChart() {
      this.chartData = [];

      let title = ["Date"];

      for (let i = 0; i < this.selections.length; i++) {
        switch (this.selections[i]) {
          case "daily_downloads":
            title.push("Daily Downloads");
            break;
          case "daily_curse_downloads":
            title.push("Daily Curse Downloads");
            break;
          case "daily_twitch_downloads":
            title.push("Daily Twitch Downloads");
            break;
          case "points":
            title.push("Points");
            break;
        }
      }

      this.chartData.push(title);
      for (let i = 0; i < csvData.length; i++) {
        let row = [csvData[i].date];

        for (let j = 0; j < this.selections.length; j++) {
          switch (this.selections[j]) {
            case "daily_downloads":
              row.push(csvData[i].dailyDownloads);
              break;
            case "daily_curse_downloads":
              row.push(csvData[i].dailyCurseDownloads);
              break;
            case "daily_twitch_downloads":
              row.push(csvData[i].dailyTwitchDownloads);
              break;
            case "points":
              row.push(csvData[i].points);
              break;
          }
        }
        this.chartData.push(row);
      }
    },
    parseCSV(file) {
      const data = [];
      const lines = file.split("\n");
      for (let line = 1; line < lines.length; line++) {
        const colums = lines[line].split(",");
        data.push({
          date: moment(colums[0], "YYYY-MM-DD").format("DD.MM.YYYY"),
          points: Number.parseFloat(colums[3], 10),
          dailyDownloads: Number.parseInt(colums[5], 10),
          dailyTwitchDownloads: Number.parseInt(colums[7], 10),
          dailyCurseDownloads: Number.parseInt(colums[8], 10)
        });
      }
      return data;
    }
  },
  watch: {
    selections(selections) {
      this.updateChart();
    }
  }
};
</script>

<style>
.chart {
  height: 70%;
}
</style>
