<template>
  <div class="wrapper" :class="{ 'nav-open': $sidebar.showSidebar }">
    <side-bar
      :background-color="sidebarBackground"
      short-title="LoRa"
      title="LoRa"
    >

      <Sensors v-if="history" class="d-none d-md-none d-lg-flex"/>

      <template slot="links">
        <sidebar-item
          :link="{
            name: 'Historie',
            icon: 'ni ni-tv-2 text-primary',
            path: '/dashboard'
          }"
        />

        <sidebar-item
            :link="{
            name: 'Aktuálně',
            icon: 'ni ni-tv-2 text-primary',
            path: '/realtime'
          }"
        />

      </template>

    </side-bar>
    <div class="main-content" :data="sidebarBackground">
<!--      <dashboard-navbar></dashboard-navbar>-->

      <div @click="toggleSidebar">
        <fade-transition :duration="200" origin="center top" mode="out-in">
          <!-- your content here -->
          <router-view></router-view>
        </fade-transition>
        <content-footer v-if="!$route.meta.hideFooter"></content-footer>
      </div>
    </div>
  </div>
</template>
<script>
  import ContentFooter from './ContentFooter.vue';
  import { FadeTransition } from 'vue2-transitions';
  import Sensors from './Sensors.vue';

  export default {
    components: {
      ContentFooter,
      FadeTransition,
      Sensors
    },
    data() {
      return {
        sidebarBackground: 'vue' //vue|blue|orange|green|red|primary
      };
    },
    computed: {
      history() {
        if(this.$route.path.includes('dashboard')) {
          return true
        } else {
          return false
        }
      }
    },
    methods: {
      toggleSidebar() {
        if (this.$sidebar.showSidebar) {
          this.$sidebar.displaySidebar(false);
        }
      }
    }
  };
</script>
<style lang="scss">
</style>
