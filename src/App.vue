<template>
  <v-app>
    <v-main>
      <v-toolbar>
        <v-toolbar-title>
          <span>CurseForge</span>
          <span class="font-weight-light ml-1">Analytics</span>
        </v-toolbar-title>

        <v-spacer></v-spacer>
        <v-btn text href="https://authors.curseforge.com/dashboard/projects" target="_blank">
          <v-icon class="mr-2">mdi-open-in-new</v-icon>Get Data
        </v-btn>
      </v-toolbar>

      <v-file-input class="mx-8 mt-8" label="Select CSV" v-model="file" accept=".csv"></v-file-input>

      <v-col cols="12">
        <v-row justify="center">
          <v-checkbox
            v-model="selections"
            label="Daily Downloads"
            value="daily_downloads"
            class="mr-8"
          ></v-checkbox>
          <v-checkbox
            v-model="selections"
            label="Daily Curse Downloads"
            value="daily_curse_downloads"
            class="mr-8"
          ></v-checkbox>
          <v-checkbox
            v-model="selections"
            label="Daily Twitch Downloads"
            value="daily_twitch_downloads"
            class="mr-8"
          ></v-checkbox>
          <v-checkbox v-model="selections" label="Points" value="points"></v-checkbox>
        </v-row>
      </v-col>
      <GChart class="chart" type="LineChart" :data="chartData" :options="chartOptions" fill-height />
    </v-main>
  </v-app>
</template>

<script>
import moment from "moment";

let csvData = [];

export default {
  name: "App",
  components: {},
  data() {
    return {
      file: null,
      chartData: [
        [
          "Date",
          "Daily Downloads",
          "Daily Twitch Downloads",
          "Daily Curse Downloads",
          "Points",
        ],
        ["", 1, 1, 1, 1],
      ],
      chartOptions: {
        curveType: "function",
        vAxis: { viewWindow: { min: 0 } },
      },
      fileName: "",
      selections: [
        "daily_downloads",
        "daily_curse_downloads",
        "daily_twitch_downloads",
        "points",
      ],
    };
  },
  methods: {
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
          dailyCurseDownloads: Number.parseInt(colums[8], 10),
        });
      }
      return data;
    },
  },
  watch: {
    selections(selections) {
      this.updateChart();
    },
    file(file) {
      if (!file) {
        return;
      }
      this.fileName = file.name;
      const reader = new FileReader();
      reader.onload = (event) => {
        csvData = this.parseCSV(event.target.result);
        this.updateChart();
      };
      reader.readAsText(file);
    },
  },
};
</script>

<style>
.chart {
  height: 70%;
}
</style>
