<template>
  <div>
    <div class="container-fluid">
      <div class="row">
        <div class="col-xl-12 mb-5 mb-xl-0 mt-2">
          <div class="mt++10">
            <Map :sensors="sensors"/>
            <p>Hodnoty jsou načítány každých 10 minut. Poslední aktualizace {{currentDate}}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
  // Charts
  import axios from "axios";
  import Map from './Map.vue';

  export default {
    components: {
      Map,
    },
    data() {
      return {
        sensors: [],
        currentDate: null,
      };
    },
    beforeDestroy() {
      clearInterval(this.polling);
    },
    created() {
      this.pollData();
    },
    computed: {

    },
    methods: {
      pollData() {
        var current_component = this;
        this.polling = setInterval(() => {
          current_component.getSensors(current_component.currentDate);
        }, 600000);
      },
      getSensors(date) {
        var current_component = this;
        this.currentDate = new Date();
        axios
        .get(
            this.$store.state.serviceUrl + "latest.json"
        )
        .then(function (response) {
          console.log(response);
          if (response.status == "200") {
            current_component.$root.$emit("updateMap", response.data);
          } else {
            alert(response.data.error);
          }
        })
        .catch(function (error) {
          console.log(error);
        });
      },
    },
    mounted() {
      this.getSensors(this.currentDate);
    }
  };
</script>
<style></style>
