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
<!--                    <b-button class="m-1" size="sm" @click="setMonth(2)">Únor</b-button>-->
                    <b-button class="m-1" size="sm" @click="setMonth(3)">Březen</b-button>
                    <b-button class="m-1" size="sm" @click="setMonth(4)">Duben</b-button>
                    <b-button class="m-1" size="sm" @click="setMonth(5)">Květen</b-button>
                    <b-button class="m-1" size="sm" @click="setMonth(6)">Červen</b-button>
                    <b-button class="m-1" size="sm" @click="setMonth(7)">Červenec</b-button>
                  </div>
                </card>
              </div>

            </div>
  <!--            <div class="row">-->
  <!--              <div class="col-xl-12 mb-12 mb-xl-0">-->
  <!--                <p>&nbsp;</p>-->
  <!--              </div>-->
  <!--            </div>-->
            <div class="row mt-2">
              <div class="col-xl-12 mb-12 mb-xl-0">
                <card type="default" header-classes="bg-transparent">
                  <div slot="header" class="row align-items-center">
                    <div class="col">
                      <h5 class="text-light text-uppercase ls-1 mb-1">Teplota</h5>
                      <h6 class="text-white mb-0">Hodnoty pro {{currentDate}}</h6>
                      <h6 class="text-light text-uppercase ls-1 mb-1">Min: {{activeSensor.measured_min}}°C, Max: {{activeSensor.measured_max}}°C</h6>
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

            <div class="row mt-2">
              <div class="col-xl-12 mb-12 mb-xl-0">
                <card type="default" header-classes="bg-transparent">
                  <div slot="header" class="row align-items-center">
                    <div class="col">
                      <h5 class="text-light text-uppercase ls-1 mb-1">Vlhkost</h5>
                      <h6 class="text-white mb-0">Hodnoty pro {{currentDate}}</h6>
                      <h6 class="text-light text-uppercase ls-1 mb-1">Min: {{activeSensor.humidity_min}}%, Max: {{activeSensor.humidity_max}}%</h6>
                    </div>
                  </div>
                  <line-chart
                      :height="350"
                      ref="humidityChart"
                      :chart-data="humidityChart.chartData"
                      :extra-options="humidityChart.extraOptions"
                  >
                  </line-chart>

                </card>
              </div>
            </div>

            <div class="row mt-2">
              <div class="col-xl-12 mb-12 mb-xl-0">
                <card type="default" header-classes="bg-transparent">
                  <div slot="header" class="row align-items-center">
                    <div class="col">
                      <h6 class="text-light text-uppercase ls-1 mb-1">Stažení dat</h6>
                      <p><a class="text-light" :href="urlData" target="_blank">Hodnoty pro vybraný den (SSV)</a></p>
                      <p><a class="text-light" :href="urlDataMonth" target="_blank">Hodnoty pro celý měsíc (ZIP/SSV)</a></p>
                    </div>
                  </div>
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
        currentMonth: this.getStartMonth(),
        currentDay: this.getStartDay(),
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
        dateSliderBefore: this.getStartDay(),
        dateSlider: this.getStartDay(),
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
        },
        humidityChart: {
          chartData: {
            datasets: [],
            labels: [],
          },
          extraOptions: chartConfigs.blueChartOptions,
        }
      };
    },
    computed: {
      urlData: function() {
        return this.getDownloadUrl('day')
      },
      urlDataMonth: function() {
        return this.getDownloadUrl('month')
      },
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
      getDownloadUrl(type) {
        let date = this.currentDate;
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
        if (type == 'day') {
          return this.$store.state.serviceUrl + "th" + sensorTitle + "/" + year + "/" + month + "/" + day + "/data.csv"
        }
        if (type == 'month') {
          return this.$store.state.serviceUrl + "th" + sensorTitle + "/" + year + "/" + month + "/data.zip"
        }
        return this.$store.state.serviceUrl + "th" + sensorTitle + "/" + year + "/" + month + "/data.zip"
      },
      getStartMonth() {
        let yesterday = new Date(new Date().setDate(new Date().getDate()-1));
        // console.log(yesterday.getMonth() + 1);
        return yesterday.getMonth() + 1;
      },
      getStartDay() {
        let yesterday = new Date(new Date().setDate(new Date().getDate()-1));
        // console.log(yesterday.getDate());
        return yesterday.getDate();
      },
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
              label: 'Teplota',
              data: this.activeSensor.measured
            }
          ],
          labels: this.activeSensor.times,
        };
        this.measuredChart.chartData = chartData;
      },
      initHumidityChart() {
        let chartData = {
          datasets: [
            {
              label: 'Vlhkost',
              data: this.activeSensor.humidity
            }
          ],
          labels: this.activeSensor.times,
        };
        this.humidityChart.chartData = chartData;
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
      cleanTheGraphs() {
        this.activeSensor.humidity_min = 'NaN';
        this.activeSensor.humidity_max = 'NaN';
        this.activeSensor.measured_min = 'NaN';
        this.activeSensor.measured_max = 'NaN';
        this.activeSensor.humidity = [];
        this.activeSensor.measured = [];
        this.activeSensor.times = [];
        this.initMeasuredChart();
        this.initHumidityChart();
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
            let humidity = [];
            let times = [];
            let i = 0;
            for (i = 0; i < (rows.length - 1); i++) {
              humidity.push(rows[i].split(';')[0])
              measured.push(rows[i].split(';')[1])
              times.push(rows[i].split(';')[3].split(' ')[1].substr(0, 5))
            }
            current_component.activeSensor.humidity_min = Math.min.apply(Math, humidity);
            current_component.activeSensor.humidity_max = Math.max.apply(Math, humidity);
            current_component.activeSensor.humidity = humidity;
            current_component.activeSensor.measured = measured;
            current_component.activeSensor.measured_min = Math.min.apply(Math, measured);
            current_component.activeSensor.measured_max = Math.max.apply(Math, measured);
            current_component.activeSensor.times = times;
            current_component.$root.$emit("sensorsListChanged", current_component.sensors);
            current_component.initMeasuredChart();
            current_component.initHumidityChart();
            if (current_component.activeSensor.measured.length < 1) {
              current_component.error = "Pro vybraný den nemám data.";
              current_component.cleanTheGraphs();
            } else {
              current_component.error = null;
            }

          } else {
            current_component.error = "Pro vybraný den nemám data.";
            current_component.cleanTheGraphs();
          }
        })
        .catch(function (error) {
          current_component.error = "Pro vybraný den nemám data: " + error;
          current_component.cleanTheGraphs();
        });
      },
      switchToSensor(index) {
        this.activeSensorId = index;
        this.getSensors(this.currentDate);
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
