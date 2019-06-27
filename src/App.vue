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

      <GChart class="chart" type="LineChart" :data="chartData" :options="chartOptions" fill-height/>
    </v-content>
  </v-app>
</template>

<script>
import moment from "moment";

export default {
  name: "App",
  components: {},
  data() {
    return {
      chartData: [
        [
          "Date",
          "Points",
          "Daily Downloads",
          "Daily Twitch Downloads",
          "Daily Curse Downloads"
        ],
        ["", 1, 1, 1, 1]
      ],
      chartOptions: {
        curveType: "function",
        vAxis: { viewWindow: { min: 0 } }
      },
      fileName: ""
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
        const data = this.parseCSV(event.target.result);
        this.chartData = [];
        this.chartData.push([
          "Date",
          "Points",
          "Daily Downloads",
          "Daily Twitch Downloads",
          "Daily Curse Downloads"
        ]);
        for (let i = 0; i < data.length; i++) {
          this.chartData.push([
            data[i].date,
            data[i].points,
            data[i].dailyDownloads,
            data[i].dailyTwitchDownloads,
            data[i].dailyCurseDownloads
          ]);
        }
      };
      reader.readAsText(files[0]);
    },
    parseCSV(file) {
      const data = [];
      const lines = file.split("\n");
      for (let line = 1; line < lines.length; line++) {
        const colums = lines[line].split(",");
        data.push({
          date: moment(colums[0], "YYYY-MM-DD").format("DD.MM.YYYY"),
          points: Number.parseFloat(colums[3], 10),
          totalDownloads: Number.parseInt(colums[5], 10),
          dailyDownloads: Number.parseInt(colums[5], 10),
          dailyTwitchDownloads: Number.parseInt(colums[7], 10),
          dailyCurseDownloads: Number.parseInt(colums[8], 10)
        });
      }
      return data;
    }
  }
};
</script>

<style>
.chart {
  height: 70%;
}
</style>
