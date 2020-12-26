<template>
  <div>
    <div id="map"></div>
  </div>
</template>

<script>
import "@/static/mapbox-gl.css";
var mapboxgl = require("mapbox-gl/dist/mapbox-gl.js");
const API_URL = process.env.API_URL;
const GRAPH_TOKEN = process.env.GRAPH_TOKEN;
const MAP_BOX_TOKEN = process.env.MAP_BOX_TOKEN;

export default {
  data() {
    return {
      route: null,
      graph: null,
    };
  },
  mounted() {},
  computed: {
    orderId() {
      return this.$route.params.id;
    },
    jwtToken() {
      return this.$route.query.key;
    },
  },
  async created() {
    let response = await fetch(
      API_URL + "/transaksi/view/" + this.$route.params.id,
      {
        method: "GET",
        headers: {
          Authorization: "Bearer " + this.jwtToken,
        },
      }
    );
    let data = await response.json();
    this.route = data.data.rute.map((value) => {
      return [parseFloat(value.longitude), parseFloat(value.latitude)];
    });

    let res_graph = await fetch(
      "https://graphhopper.com/api/1/route?key=" + GRAPH_TOKEN,
      {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          instructions: false,
          points_encoded: false,
          elevation: true,
          points: this.route,
          vehicle: "car",
        }),
      }
    );

    this.graph = await res_graph.json();

    this.loadMap(this.graph);
  },
  methods: {
    loadMap(graph) {
      mapboxgl.accessToken = MAP_BOX_TOKEN;
      var mapbox = new mapboxgl.Map({
        container: "map",
        style: "mapbox://styles/mapbox/streets-v11",
        center: graph.paths[0].points.coordinates[0],
        zoom: 10,
      });

      this.route.forEach((value) => {
        new mapboxgl.Marker().setLngLat(value).addTo(mapbox);
      });

      mapbox.on("load", function () {
        mapbox.addSource("route", {
          type: "geojson",
          data: {
            type: "Feature",
            properties: {},
            geometry: graph.paths[0].points,
          },
        });
        mapbox.addLayer({
          id: "route",
          type: "line",
          source: "route",
          layout: {
            "line-join": "round",
            "line-cap": "round",
          },
          paint: {
            "line-color": "#fcba03",
            "line-width": 6,
          },
        });
      });
    },
  },
};
</script>

<style scoped>
#map {
  width: 360px;
  height: 740px;
}
</style>