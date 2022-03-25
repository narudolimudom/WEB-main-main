<template>
  <div class="small">
    <h3>{{ "ระดับ : " + this.$store.getters.GetNameOFzone }}</h3>
    <LineChart
      :labelchart="labelchart"
      :label="dataLabel"
      :data="dataChart"
      :options="{ responsive: true, maintainAspectRatio: false }"
      :chartcolor="chartcolor"
      :anotherdata="anotherData"
      :anotherlabel="anotherLebel"
      :anothercolor="anotherColor"
    ></LineChart>

    <div class="btn pt-2">
      <h4>จัดลำดับตามจำนวน</h4>

      <v-radio-group row dense class="radio">
        <v-radio
          class="ra"
          @click="SelectKeyForChart(index)"
          :label="key"
          :value="key"
          :color="getColorRadio(index)"
          v-for="(key, index) in this.$store.getters.GetKeyList"
          :key="'A' + index"
        ></v-radio>
      </v-radio-group>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import LineChart from "./BarCharts.vue";
export default {
  components: {
    LineChart,
  },

  computed: {
    KeyForChart() {
      return this.$store.getters.GetKeyForChart;
    },
    WatchRegionSelect() {
      return this.$store.getters.GetRegionSelect;
    },
  },
  watch: {
    KeyForChart() {
      this.changeData();
    },
    WatchRegionSelect(region) {
      if (region == "All") {
        this.HospitalByRegion = this.allData;
      } else {
        this.HospitalByRegion = this.allData.filter(
          (hospital) => hospital.region == region
        );
      }
      this.changeData();
    },
  },

  data() {
    return {
      allData: [],
      labelchart: null,
      change: null,
      dataChart: [],
      sortData: null,
      dataLabel: [],
      chartcolor: null,
      HospitalByRegion: [],
      anotherData: [],
      anotherLebel: [],
      anotherColor: [],
      NameOFzone: "ระดับประเทศ",
      JsonColor: [],
    };
  },
  methods: {
    changeData() {
      var KeyList = [];

      KeyList.push(Object.keys(this.HospitalByRegion[0].show_data));
      this.labelchart = KeyList[0][this.$store.getters.GetKeyForChart];
      this.sortData = this.HospitalByRegion.sort(
        (a, b) => b.show_data[this.labelchart] - a.show_data[this.labelchart]
      );

      this.dataLabel = this.sortData.map((item) => item.name);

      this.dataChart = this.sortData.map(
        (item) => item.show_data[this.labelchart]
      );

      this.chartcolor = this.JsonColor[this.labelchart];
      this.anotherData = [];
      this.anotherLebel = [];
      this.anotherColor = [];
      for (let index = 0; index < KeyList[0].length; index++) {
        if (this.labelchart != KeyList[0][index]) {
          this.anotherData.push(
            this.sortData.map((item) => item.show_data[KeyList[0][index]])
          );
          this.anotherLebel.push(KeyList[0][index]);

          this.anotherColor.push(this.JsonColor[KeyList[0][index]]);
        }
      }
    },
    SelectKeyForChart(index) {
      this.$store.state.KeyForChart = index;
    },
    getColorRadio(index) {
      return this.JsonColor[this.$store.getters.GetKeyList[index]];
    },
  },
  mounted() {
    this.$store.dispatch("feedDatatToKeyForChart", 0); //inital show data

    axios.get(this.$store.getters.GetURLdata).then((resp) => {
      this.allData = resp.data.info;
      this.JsonColor = resp.data.colors;
      this.$store.dispatch(
        "feedDataToKeyList",
        Object.keys(resp.data.info[0].show_data)
      );

      this.HospitalByRegion = resp.data.info;

      if (this.$store.getters.GetRegionSelect != "All") {
        this.HospitalByRegion = this.allData.filter(
          (hospital) => hospital.region == this.$store.getters.GetRegionSelect
        );
      }
      console.log(this.$store.getters.GetNameOFzone);
      this.changeData();
    });
  },
};
</script>

<style scoped>
.radio >>> label {
  font-size: 12px;

  padding-left: 0px;
}

.btn {
  display: flex;
  flex-direction: column;
  align-items: center;
}
</style>
