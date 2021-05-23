<template xmlns:>
  <div id="mapapp" :class="[$options.name]">
    <!-- app map -->
    <vl-map v-if="mapVisible" class="map" ref="map" :load-tiles-while-animating="true" :load-tiles-while-interacting="true"
            @postcompose="onMapPostCompose"
            data-projection="EPSG:4326" @mounted="onMapMounted">
      <!-- map view aka ol.View -->
      <vl-view ref="view" :center.sync="center" :zoom.sync="zoom" :rotation.sync="rotation"></vl-view>

      <!-- base layers -->
      <vl-layer-tile v-for="layer in baseLayers" :key="layer.name" :id="layer.name" :visible="layer.visible">
        <component :is="'vl-source-' + layer.name" v-bind="layer"></component>
      </vl-layer-tile>
      <!--// base layers -->

      <!-- other layers from config -->
      <component v-for="layer in layers" :is="layer.cmp" v-if="layer.visible" :key="layer.id" v-bind="layer">
        <!-- add vl-source-* -->
        <component :is="layer.source.cmp" v-bind="layer.source">
          <!-- add static features to vl-source-vector if provided -->
          <vl-feature v-if="layer.source.staticFeatures && layer.source.staticFeatures.length"
                      v-for="feature in layer.source.staticFeatures" :key="feature.id"
                      :id="feature.id" :properties="feature.properties">
            <component :is="geometryTypeToCmpName(feature.geometry.type)" v-bind="feature.geometry"></component>
            <vl-style-box>
              <vl-style-fill color="white"></vl-style-fill>
              <vl-style-stroke :color="feature.properties.stroke_color"></vl-style-stroke>
              <vl-style-text v-if="zoom > 14 && zoom < 16" :text="feature.properties.label" font="10px monospace"></vl-style-text>
              <vl-style-text v-if="zoom > 15" :text="feature.properties.label_ext" font="10px monospace"></vl-style-text>
            </vl-style-box>
          </vl-feature>

          <!-- add inner source if provided (like vl-source-vector inside vl-source-cluster) -->
          <component v-if="layer.source.source" :is="layer.source.source.cmp" v-bind="layer.source.source">
            <!-- add static features to vl-source-vector if provided -->
            <vl-feature v-if="layer.source.source.staticFeatures && layer.source.source.staticFeatures.length"
                        v-for="feature in layer.source.source.staticFeatures" :key="feature.id"
                        :id="feature.id" :properties="feature.properties" @click="showInfo(feature.properties)">
              <component :is="geometryTypeToCmpName(feature.geometry.type)" v-bind="feature.geometry"></component>
            </vl-feature>
          </component>
        </component>
        <!--// vl-source-* -->

        <!-- add style components if provided -->
        <!-- create vl-style-box or vl-style-func -->
        <component v-if="layer.style" v-for="(style, i) in layer.style" :key="i" :is="style.cmp" v-bind="style">
          <!-- create inner style components: vl-style-circle, vl-style-icon, vl-style-fill, vl-style-stroke & etc -->
          <component v-if="style.styles" v-for="(st, cmp) in style.styles" :key="cmp" :is="cmp" v-bind="st">
            <!-- vl-style-fill, vl-style-stroke if provided -->
            <vl-style-fill v-if="st.fill" v-bind="st.fill"></vl-style-fill>
            <vl-style-stroke v-if="st.stroke" v-bind="st.stroke"></vl-style-stroke>
          </component>
        </component>
        <!--// style -->
      </component>
      <!--// other layers -->

      <!-- interactions -->
<!--      <vl-interaction-select-->
<!--          :features.sync="selectedFeatures">-->
<!--        <template slot-scope="selection">-->
<!--          &lt;!&ndash; select styles &ndash;&gt;-->
<!--          <vl-style>-->
<!--            <vl-style-stroke-->
<!--                color="#423e9e"-->
<!--                :width="7" />-->
<!--            <vl-style-fill-->
<!--                :color="[254, 178, 76, 0.7]"/>-->
<!--            <vl-style-circle :radius="5">-->
<!--              <vl-style-stroke-->
<!--                  color="#423e9e"-->
<!--                  :width="7"/>-->
<!--              <vl-style-fill-->
<!--                  :color="[254, 178, 76, 0.7]"/>-->
<!--            </vl-style-circle>-->
<!--          </vl-style>-->
<!--          <vl-style :z-index="1">-->
<!--            <vl-style-stroke-->
<!--                color="#d43f45"-->
<!--                :width="2"/>-->
<!--            <vl-style-circle :radius="5">-->
<!--              <vl-style-stroke-->
<!--                  color="#d43f45"-->
<!--                  :width="2"/>-->
<!--            </vl-style-circle>-->
<!--          </vl-style>-->
<!--          &lt;!&ndash;// select styles &ndash;&gt;-->

<!--          &lt;!&ndash; selected feature popup &ndash;&gt;-->
<!--          <vl-overlay-->
<!--              v-for="feature in select.features"-->
<!--              :key="feature.id"-->
<!--              :id="feature.id"-->
<!--              class="feature-popup"-->
<!--              :position="pointOnSurface(feature.geometry)"-->
<!--              :auto-pan="true"-->
<!--              :auto-pan-animation="{ duration: 300 }">-->
<!--            <template slot-scope="popup">-->
<!--              <section class="card">-->
<!--                <header class="card-header">-->
<!--                  <p class="card-header-title">-->
<!--                    Feature ID {{ feature.id }}-->
<!--                  </p>-->
<!--                  <a class="card-header-icon" title="Close"-->
<!--                     @click="selectedFeatures = selectedFeatures.filter(f => f.id !== feature.id)">-->
<!--                    <b-icon icon="close"></b-icon>-->
<!--                  </a>-->
<!--                </header>-->
<!--                <div class="card-content">-->
<!--                  <div class="content">-->
<!--                    <p>-->
<!--                      Overlay popup content for Feature with ID <strong>{{ feature.id }}</strong>-->
<!--                    </p>-->
<!--                    <p>-->
<!--                      Popup: {{ JSON.stringify(popup) }}-->
<!--                    </p>-->
<!--                    <p>-->
<!--                      Feature: {{ JSON.stringify({ id: feature.id, properties: feature.properties }) }}-->
<!--                    </p>-->
<!--                  </div>-->
<!--                </div>-->
<!--              </section>-->
<!--            </template>-->
<!--          </vl-overlay>-->
<!--          &lt;!&ndash;// selected popup &ndash;&gt;-->
<!--        </template>-->
<!--      </vl-interaction-select>-->

    </vl-map>

  </div>
</template>

<script>

  import {kebabCase, camelCase} from 'lodash'
  import {createStyle} from 'vuelayers/lib/ol-ext'
  import ScaleLine from 'ol/control/ScaleLine'
  import FullScreen from 'ol/control/FullScreen'
  import ZoomSlider from 'ol/control/ZoomSlider'

  const methods = {
    camelCase,
    geometryTypeToCmpName(type) {
      return 'vl-geom-' + kebabCase(type)
    },
    showInfo(properties) {
      console.log(properties);
    },
    selectFilter(feature) {
      return ['position-feature'].indexOf(feature.getId()) === -1
    },
    onMapPostCompose({vectorContext, frameState}) {
      if (!this.$refs.marker || !this.$refs.marker.$feature) {
        return
      }

      const feature = this.$refs.marker.$feature
      if (!feature.getGeometry() || !feature.getStyle()) {
        return
      }

      const flashGeom = feature.getGeometry().clone()
      const duration = feature.get('duration')
      const elapsed = frameState.time - feature.get('start')
      const elapsedRatio = elapsed / duration
      const radius = easeInOut(elapsedRatio) * 35 + 5
      const opacity = easeInOut(1 - elapsedRatio)
      const fillOpacity = easeInOut(0.5 - elapsedRatio)

      vectorContext.setStyle(createStyle({
        imageRadius: radius,
        fillColor: [119, 170, 203, fillOpacity],
        strokeColor: [119, 170, 203, opacity],
        strokeWidth: 2 + opacity,
      }))

      vectorContext.drawGeometry(flashGeom)
      vectorContext.setStyle(feature.getStyle()(feature)[0])
      vectorContext.drawGeometry(feature.getGeometry())

      if (elapsed > duration) {
        feature.set('start', Date.now())
      }

      this.$refs.map.render()
    },
    onMapMounted() {
      // now ol.Map instance is ready and we can work with it directly
      this.$refs.map.$map.getControls().extend([
        new ScaleLine(),
        new FullScreen(),
        new ZoomSlider(),
      ])
    },
    getStrokeColor(st) {
      const currentdate = new Date();
      const sensorTime = Date.parse(st);
      const difference = Math.abs(sensorTime - currentdate);
      console.log(difference);
      if (difference > 1000 * 60 * 60 * 10) {
        return "red";
      } else {
        return "green";
      }
    },
    updateMap: function (sensors) {
      // this.drawCircles([18.09250, 18.09350],[49.84162, 49.84162], [100, 200])
      let features = [];
      for (let i = 0; i < sensors.length; i++) {
        let id = i + 1;
        if (id < 10) {
          id = "0" + id;
        }
        let feature = {
          type: 'Feature',
          id: 'sensor-error-' + i,
          geometry: {
            type: 'Circle',
            coordinates: [sensors[i].lon, sensors[i].lat],
            radius: 120,
          },
          properties: {
            "label": sensors[i].temp + '°C\n' + sensors[i].time.substring(11, 16),
            "label_ext": "ID: " + id + "\n" + sensors[i].temp + '°C\n' + sensors[i].hum + '%\n' + sensors[i].time.substring(11, 16),
            "hum": sensors[i].hum,
            "bat": sensors[i].bat,
            "temp": sensors[i].temp,
            "time": sensors[i].time,
            "stroke_color": this.getStrokeColor(sensors[i].time),
          }
        }
        features.push(feature);
      }
      let source = {
        cmp: 'vl-source-vector',
        staticFeatures: features
      }
      this.layers[0].source = source;
      this.layers[0].visible = true;
    }
  }
  export default {
    name: 'vld-demo-app',
    props: {
      sensor: {
        type: Object,
        default: null,
        description: "Data about current sensor."
      },
      maptype: {
        type: String,
        default: "history",
        description: "Type of the map."
      }
    },
    methods,
    computed: {
      label(properties) {
        return properties.temp;
      }
    },
    mounted: function () {
      this.updateMap([]);
      this.$root.$on("updateMap", (sensors) => {
        this.updateMap(sensors);
      });
    },
    data() {
      return {
        statType: 'avg',
        name: '',
        nameState: null,
        selectedFeatures: [],
        center: [18.09168, 49.84150],
        zoom: 15,
        rotation: 0,
        clickCoordinate: undefined,
        deviceCoordinate: undefined,
        mapPanel: {
          tab: 'draw',
        },
        panelOpen: true,
        mapVisible: true,
        // base layers
        baseLayers: [
          {
            name: 'osm',
            title: 'OpenStreetMap',
            visible: true,
          }
        ],
        // layers config
        layers: [
          // Circles
          {
            id: 'circles',
            title: 'Circles',
            cmp: 'vl-layer-vector',
            visible: false,
            source: {
              cmp: 'vl-source-vector',
              staticFeatures: [],
            },
            style: [
              {
                cmp: 'vl-style-box',
                styles: {
                  'vl-style-fill': {
                    color: [255, 0, 0, 0],
                  },
                  'vl-style-stroke': {
                    color: '#219e46',
                    width: 2,
                  },
                },
              },
            ],
          },
        ],
      }
    },
  }
</script>

<style lang="sass">
  @import ~bulma/sass/utilities/_all

  #mapapp
    width: 100%
    height: 600px
    margin: 0
    padding: 10

  #modal-save-area
    position: relative
    top: 10px

  .vld-demo-app
    position: relative

    .map
      height: 100%
      width: 100%

    .map-panel
      padding: 0
      position: absolute
      left: 20%
      transform: translateX(-20%)

      .panel-heading
        box-shadow: 0 .25em .5em transparentize($dark, 0.8)

      .panel-content
        background: $white
        box-shadow: 0 .25em .5em transparentize($dark, 0.8)

      .panel-block
        &.draw-panel
          .buttons
            .button
              display: block
              flex: 1 1 100%

      +tablet()
        position: absolute
        top: 5px
        right: 40px
        max-height: 500px
        width: 32em

    .base-layers-panel
      position: absolute
      left: 50%
      bottom: 20px
      transform: translateX(-50%)

    .feature-popup
      position: absolute
      left: -50px
      bottom: 12px
      width: 20em
      max-width: none

      &:after, &:before
        top: 100%
        border: solid transparent
        content: ' '
        height: 0
        width: 0
        position: absolute
        pointer-events: none
      &:after
        border-top-color: white
        border-width: 10px
        left: 48px
        margin-left: -10px
      &:before
        border-top-color: #cccccc
        border-width: 11px
        left: 48px
        margin-left: -11px

      .card-content
        max-height: 20em
        overflow: auto

      .content
        word-break: break-all
</style>
