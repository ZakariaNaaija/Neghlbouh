<template>
  <div>
    <v-row>
      <v-col cols="12" v-if="loaded">
        <v-row
          align="center"
          class="mt-10 text-center"
          justify="center"
          v-if="alertMe"
        >
          <v-col align-self="center" cols="4" justify="center">
            <v-alert dense text type="error"
              >خطر في منطقتك الرجاء إلتزام المنزل
            </v-alert>
          </v-col>
        </v-row>
        <v-row justify="center">
          <v-col md="9" cols="11">
            <div class="mapboxgl-map" id="map">
              <MglMap
                :accessToken="accessToken"
                :center.sync="center"
                :mapStyle.sync="mapStyle"
                :maxBounds="maxBounds"
                :zoom="zoom"
                container="map"
              >
                <MglGeolocateControl
                  position="top-right"
                  show="true"
                  class="fas fa-crosshairs"
                />
                <MglScaleControl />
                <MglMarker
                  :coordinates="myCoordinate"
                  :key="mykey"
                  color="red"
                  v-if="visible"
                />
                <MglGeojsonLayer
                  :key="index + 30"
                  :layer="getGeoJsonLayer(index)"
                  :layerId="index.toString()"
                  :source="getGeoJsonSource(index, zone)"
                  :sourceId="index.toString()"
                  v-for="(zone, index) in miniZones"
                >
                </MglGeojsonLayer>
                <MglGeojsonLayer
                  :key="index + 500"
                  :layer="getTextLayer()"
                  :layerId="(index + 500).toString()"
                  :source="getGeoJsonSource(index + 500, zone)"
                  :sourceId="(index + 500).toString()"
                  v-for="(zone, index) in miniZones"
                >
                </MglGeojsonLayer>
              </MglMap>
            </div>
          </v-col>
        </v-row>
        <v-row justify="center">
          <v-col cols="10" align-self="center">
            <v-row>
              <v-col cols="4">
                <div class="text-center fill-height">
                  <v-sheet color="red lighten-5 " class="fill-height">
                    اقل 20 مواطن</v-sheet
                  >
                </div>
              </v-col>
              <v-col cols="4">
                <div class="text-center fill-height">
                  <v-sheet color="red lighten-4"> بين 20 و 50 مواطن</v-sheet>
                </div>
              </v-col>
              <v-col cols="4">
                <div class="text-center fill-height">
                  <v-sheet color="red lighten-2" class="fill-height">
                    اكثر من 50</v-sheet
                  >
                </div>
              </v-col>
            </v-row>
          </v-col>
        </v-row>
      </v-col>
    </v-row>
    <v-row align="center" justify="center" v-if="!loaded">
      <v-col align="center" justify="center">
        <v-row>
          <v-col>
            <v-progress-circular
              :width="5"
              color="black"
              indeterminate
              size="50"
            >
            </v-progress-circular>
          </v-col>
        </v-row>
        <v-row>
          <v-col>
            <span class="font">
              لحظة برك
            </span>
          </v-col>
        </v-row>
      </v-col>
    </v-row>
  </div>
</template>

<script>
import Mapbox from "mapbox-gl";
import gps from "@/services/GpsService";
import cord from "@/store/coordonnees.json";
import {
  MglGeojsonLayer,
  MglGeolocateControl,
  MglMap,
  MglMarker,
  MglScaleControl
} from "vue-mapbox";

export default {
  name: "LiveMap",
  components: {
    MglMap,
    MglMarker,
    MglGeojsonLayer,
    MglGeolocateControl,
    MglScaleControl
  },
  data() {
    return {
      geoJson: "",
      loaded: false,
      alertMe: false,
      visible: false,
      mykey: "100",
      numb: "0",
      myCoordinate: [10.5449929, 34.7267589],
      accessToken:
        "pk.eyJ1Ijoic2FtaWVsbGV1Y2giLCJhIjoiY2s4ZmYxanp5MDA5MDNmcWowY3FuZm1tbSJ9.neFuBaRgOGr8khOj2FGweA",
      mapStyle: "mapbox://styles/mapbox/light-v10",
      center: [10.5375, 35.2],
      zoom: 6,
      maxBounds: [
        [6.5, 28.8869],
        [12.5375, 38.1]
      ],
      Zones: cord.Zones,
      miniZones: [[], [], []]
    };
  },
  methods: {
    getGeoJsonSource(idd, cord) {
      let createdFeatures = [];
      for (let i = 0; i < cord.length; i++) {
        createdFeatures.push({
          type: "Feature",
          geometry: {
            type: "Point",
            coordinates: cord[i].cord
          },
          properties: { title: cord[i].name + "\n" + cord[i].countPeople }
        });
      }
      return {
        type: "geojson",
        data: {
          id: idd.toString(),
          type: "FeatureCollection",
          features: createdFeatures
        }
      };
    },
    getGeoJsonLayer(nb) {
      let opacity = 0;
      if (nb == "faible") {
        opacity = 0.1;
      } else if (nb == "moyen") {
        opacity = 0.3;
      } else {
        opacity = 0.5;
      }
      return {
        type: "circle",
        paint: {
          "circle-color": "red",
          "circle-radius": {
            stops: [
              [0, 0],
              [20, 18000]
              //distance here
            ],
            base: 2
          },
          "circle-stroke-color": "white",
          "circle-opacity": opacity,
          "circle-pitch-scale": "viewport"
        }
      };
    },
    getTextLayer() {
      return {
        type: "symbol",
        minzoom: 9,
        maxzoom: 0,
        layout: {
          "text-field": ["get", "title"],
          "text-font": ["DIN Offc Pro Medium", "Arial Unicode MS Bold"],
          "text-size": 13
        }
      };
    }
  },
  created() {
    // We need to set mapbox-gl library here in order to use it in template
    this.mapbox = Mapbox;
    //to be removed and changed to api call
  },
  async mounted() {
    if (this.mapbox.getRTLTextPluginStatus() !== "loaded") {
      this.mapbox.setRTLTextPlugin(
        "https://api.mapbox.com/mapbox-gl-js/plugins/mapbox-gl-rtl-text/v0.2.3/mapbox-gl-rtl-text.js",
        null,
        true // Lazy load the plugin
      );
    }
    let Zones = await gps.requestGPS();
    this.miniZones = Zones.data.data;

    this.loaded = true;

    //every 10 second request the api !
  }
};
</script>
<style>
#map {
  border: 3px red solid;
  width: 100%;
  height: 900px;
}
.mapboxgl-ctrl-geolocate {
  background-image: url("../assets/target.svg") !important;
}
.font {
  font-size: 1.2em;
  font-weight: bold;
  font-family: Cairo;
}
</style>
