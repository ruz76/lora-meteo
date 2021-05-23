<template>
  <div>
<!--    <div>-->
<!--      <base-header type="gradient-success" class="pb-20 pt-5 pt-md-5">-->
<!--      </base-header>-->
<!--    </div>-->
    <div class="container-fluid mt-100">
      <!--Tables-->
      <div class="row">
        <div class="col-xl-12 mb-5 mb-xl-0">
          <!--Charts-->
          <div class="container-fluid mt-2">
            <div class="row">
              <div class="col-xl-12 mb-12 mb-xl-0">
                <card type="default" header-classes="bg-transparent">
                  <h5 class="text-light text-uppercase ls-1 mb-1">Senzor: {{sensors[activeSensorId].title}} Datum: {{currentDate}}</h5>
                  <p class="text-white"></p>
                  <p class="text-white"><span v-if="error != null">{{error}}</span>&nbsp;</p>
                  <base-slider
                      ref="dateSlider"
                      v-model="dateSlider"
                      :range="dateSliderRangeComputed"
                      :step="1"
                      :onchange="changeDate()"
                      :key="sliderkey"
                  />
                  <div>
                    <b-button size="sm" @click="setMonth(2)">Únor</b-button>
                    <b-button size="sm" @click="setMonth(3)">Březen</b-button>
                    <b-button size="sm" @click="setMonth(4)">Duben</b-button>
                    <b-button size="sm" @click="setMonth(5)">Květen</b-button>
                  </div>
                </card>
              </div>

            </div>
            <div class="row">
              <div class="col-xl-12 mb-12 mb-xl-0">
                <p>&nbsp;</p>
              </div>
            </div>
            <div class="row">
              <div class="col-xl-12 mb-12 mb-xl-0">
                <card type="default" header-classes="bg-transparent">
                  <div slot="header" class="row align-items-center">
                    <div class="col">
                      <h6 class="text-light text-uppercase ls-1 mb-1">Teplota</h6>
                      <h5 class="h3 text-white mb-0">Hodnoty pro {{currentDate}}</h5>
                    </div>
                  </div>
                  <line-chart
                      :height="350"
                      ref="measuredChart"
                      :chart-data="measuredChart.chartData"
                      :extra-options="measuredChart.extraOptions"
                  >
                  </line-chart>

                </card>
              </div>

            </div>
            <!-- End charts-->
          </div>
        </div>
        <!--End tables-->

      </div>

    </div>
  </div>
</template>
<script>
  // Charts
  import axios from "axios";
  import * as chartConfigs from '@/components/Charts/config';
  import LineChart from '@/components/Charts/LineChart';

  // Tables
  // import Map from './Map.vue';

  export default {
    components: {
      LineChart
    },
    data() {
      return {
        error: null,
        sliderkey: 1,
        currentTimeLevelIndex: 1,
        currentTimeLevel: "month",
        timelevels: ["year", "month", "day", "hour", "minute"],
        currentYear: 2021,
        currentMonth: 5,
        currentDay: 15,
        currentHour: 12,
        currentMinute: 30,
        currentSecond: 30,
        sensors: [
            { "title": "01", "info": "Nevím"},
            { "title": "02", "info": "Nevím"},
            { "title": "03", "info": "Nevím"},
            { "title": "04", "info": "Nevím"},
            { "title": "05", "info": "Nevím"},
            { "title": "06", "info": "Nevím"},
            { "title": "07", "info": "Nevím"},
            { "title": "08", "info": "Nevím"},
            { "title": "09", "info": "Nevím"},
            { "title": "10", "info": "Nevím"}
          ],
        activeSensorId: 0,
        activeSensor: {title: "", info: ""},
        statType: 'avg',
        dateSliderBefore: 15,
        dateSlider: 15,
        timeLevelSliderBefore: 0,
        timeLevelSlider: 0,
        timeLevelSliderRange: {min: 0, max: 4},
        distancesErrorChart: {
          chartData: {
            datasets: [],
            labels: [],
          },
          extraOptions: chartConfigs.blueChartOptions,
        },
        measuredChart: {
          chartData: {
            datasets: [],
            labels: [],
          },
          extraOptions: chartConfigs.blueChartOptions,
        }
      };
    },
    computed: {
      // a computed getter
      dateSliderRangeComputed: function () {
        switch (this.currentTimeLevel) {
          case "year":
            return {min: 1, max: 12};
          case "month":
            return {min: 1, max: 31};
          case "day":
            return {min: 0, max: 23};
          case "hour":
            return {min: 0, max: 59};
          case "minute":
            return {min: 0, max: 59};
        }
      },
      currentDate: function () {
        switch (this.currentTimeLevel) {
          case "year":
            return this.currentYear + "-" + Math.round(this.dateSlider);
          case "month":
            return this.currentYear + "-" + this.currentMonth + "-" + Math.round(this.dateSlider);
          case "day":
            return this.currentYear + "-" + this.currentMonth + "-" + this.currentDay + " " + Math.round(this.dateSlider);
          case "hour":
            return this.currentYear + "-" + this.currentMonth + "-" + this.currentDay + " " + this.currentHour + ":" + Math.round(this.dateSlider);
          case "minute":
            return this.currentYear + "-" + this.currentMonth + "-" + this.currentDay + " " + this.currentHour + ":" + this.currentMinute + ":" + Math.round(this.dateSlider);
          default:
            return this.currentYear + "-" + this.currentMonth + "-" + Math.round(this.dateSlider);
        }
      },
      distanceErrorDataComputed: function () {
        let cdata = this.activeSensor.avg_distance_error;
        switch (this.$store.state.statType) {
          case 'avg':
            cdata = this.activeSensor.avg_distance_error;
            break;
          case 'min':
            cdata = this.activeSensor.min_distance_error;
            break;
          case 'max':
            cdata = this.activeSensor.max_distance_error;
            break;
          default:
            cdata = this.activeSensor.avg_distance_error;
        }
        let chartData = {
          datasets: [
            {
              label: 'Distance error',
              data: cdata
            }
          ],
          labels: this.activeSensor.times,
        };
        return chartData;
      }
    },
    methods: {
      timeLevelStatus(index) {
        // console.log('AAAA', index, this.currentTimeLevel);
        return this.currentTimeLevelIndex === index ? 'success' : 'secondary';
      },
      statTypeStatus(type) {
        // console.log('BBBB', type, this.statType);
        return this.statType === type ? 'success' : 'secondary';
      },
      switchToTimeLevel(index) {
        // console.log(index);
        this.currentTimeLevelIndex = index;
        this.currentTimeLevel = this.timelevels[index];
        this.sliderkey = this.sliderkey + 1;
        switch (this.currentTimeLevel) {
          case "year":
            this.dateSlider = this.currentMonth;
            break;
          case "month":
            this.dateSlider = this.currentDay;
            break;
          case "day":
            this.dateSlider = this.currentHour;
            break;
          case "hour":
            this.dateSlider = this.currentMinute;
            break;
          case "minute":
            this.dateSlider = this.currentSecond;
            break;
          default:
            this.dateSlider = this.currentDay;
        }
      },
      setMonth(month) {
        this.currentMonth = month;
        this.getSensors(this.currentDate);
      },
      setStat(type) {
        this.statType = type;
        this.$store.commit('setStat', type);
        this.$root.$emit("updateMap", type);
      },
      initDistancesErrorChart() {
        let cdata = this.activeSensor.avg_distance_error;
        switch (this.statType) {
          case 'avg':
            cdata = this.activeSensor.avg_distance_error;
            break;
          case 'min':
            cdata = this.activeSensor.min_distance_error;
            break;
          case 'max':
            cdata = this.activeSensor.max_distance_error;
            break;
          default:
            cdata = this.activeSensor.avg_distance_error;
        }
        let chartData = {
          datasets: [
            {
              label: 'Distance error',
              data: cdata
            }
          ],
          labels: this.activeSensor.times,
        };
        this.distancesErrorChart.chartData = chartData;
      },
      initMeasuredChart() {
        let chartData = {
          datasets: [
            {
              label: 'Temperature',
              data: this.activeSensor.measured
            }
          ],
          labels: this.activeSensor.times,
        };
        this.measuredChart.chartData = chartData;
      },
      changeTimeLevel() {
        if (this.timeLevelSlider !== this.timeLevelSliderBefore) {
          this.timeLevelSliderBefore = this.timeLevelSlider;
          this.switchToTimeLevel(Math.round(this.timeLevelSlider));
        }
      },
      changeDate() {
        //console.log(this.dateSlider);
        if (this.dateSlider !== this.dateSliderBefore) {
          this.dateSliderBefore = this.dateSlider;
          let datetocheck = null;
          // console.log(this.currentTimeLevel);
          switch (this.currentTimeLevel) {
            case "year":
              datetocheck = this.currentYear + "-" + Math.round(this.dateSlider) + "-01";
              this.currentMonth = Math.round(this.dateSlider);
              break;
            case "month":
              datetocheck = this.currentYear + "-" + this.currentMonth + "-" + Math.round(this.dateSlider);
              this.currentDay = Math.round(this.dateSlider);
              break;
            case "day":
              datetocheck = this.currentYear + "-" + this.currentMonth + "-" + this.currentDay + " " + Math.round(this.dateSlider) + ":00:00";
              this.currentHour = Math.round(this.dateSlider);
              break;
            case "hour":
              datetocheck = this.currentYear + "-" + this.currentMonth + "-" + this.currentDay + " " + this.currentHour + ":" + Math.round(this.dateSlider) + ":00";
              this.currentMinute = Math.round(this.dateSlider);
              break;
            case "minute":
              datetocheck = this.currentYear + "-" + this.currentMonth + "-" + this.currentDay + " " + this.currentHour + ":" + this.currentMinute + ":" + Math.round(this.dateSlider);
              this.currentSecond = Math.round(this.dateSlider);
              break;
            default:
              datetocheck = this.currentYear + "-" + this.currentMonth + "-" + Math.round(this.dateSlider);
          }
          // console.log(datetocheck);
          this.getSensors(this.currentDate);
        }
      },
      getSensors(date) {
        var current_component = this;
        // console.log(date);
        let year = date.split('-')[0];
        let month = parseInt(date.split('-')[1]);
        let day = parseInt(date.split('-')[2]);
        if (month < 10) {
          month = '0' + month;
        }
        if (day < 10) {
          day = '0' + day;
        }
        let sensorTitle = this.sensors[this.activeSensorId].title;
        axios
        .get(
            this.$store.state.serviceUrl + "th" + sensorTitle + "/" + year + "/" + month + "/" + day + "/data.csv"
        )
        .then(function (response) {
          //console.log(response);
          if (response.status == "200") {
            // console.log(response.data);
            let rows = response.data.split('\n');
            let measured = [];
            let times = [];
            let i = 0;
            for (i = 0; i < (rows.length - 1); i++) {
              measured.push(rows[i].split(';')[1])
              times.push(rows[i].split(';')[3].split(' ')[1].substr(0, 5))
            }
            current_component.activeSensor.measured = measured;
            current_component.activeSensor.times = times;
            // current_component.sensors = response.data.sensors;
            // current_component.activeSensor = current_component.sensors[current_component.activeSensorId];
            // current_component.$root.$emit(
            //     "sensorChanged",
            //     current_component.activeSensor
            // );
            current_component.$root.$emit("sensorsListChanged", current_component.sensors);
            // //current_component.initDistancesErrorChart();
            current_component.initMeasuredChart();
            if (current_component.activeSensor.measured.length < 1) {
              current_component.error = "Pro vybraný den nemám data.";
              current_component.activeSensor.measured = [];
              current_component.activeSensor.times = [];
              current_component.initMeasuredChart();
            } else {
              current_component.error = null;
            }

          } else {
            // alert(response.data.error);
            // current_component.error = response.data.error;
            current_component.error = "Pro vybraný den nemám data.";
            current_component.activeSensor.measured = [];
            current_component.activeSensor.times = [];
            current_component.initMeasuredChart();
          }
        })
        .catch(function (error) {
          // console.log(error);
          // current_component.error = error;
          // let a = error;
          current_component.error = "Pro vybraný den nemám data: " + error;
          current_component.activeSensor.measured = [];
          current_component.activeSensor.times = [];
          current_component.initMeasuredChart();
        });
      },
      switchToSensor(index) {
        this.activeSensorId = index;
        this.getSensors(this.currentDate);
        // this.activeSensor = this.sensors[index];
        // if (this.activeSensor.measured.length < 1) {
        //   this.error = "There are no data for selected time.";
        // } else {
        //   this.error = null;
        // }
        // // this.initDistancesErrorChart();
        // this.initMeasuredChart();
        // //console.log("STS", this.activeSensor);
      }
    },
    mounted() {
      this.$store.commit('setMapMove', true);
      this.getSensors(this.currentDate);
      this.$root.$on("switchToSensor", (si) => {
        this.switchToSensor(si);
      });
      this.$root.$on("statChanged", (type) => {
        this.statType = type;
        this.initDistancesErrorChart();
      });
    }
  };
</script>
<style></style>
