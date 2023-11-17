<script setup>
import * as L from "leaflet";
import { onMounted } from "vue";
import "leaflet/dist/leaflet.css";
import * as d3 from "d3";
import sectores_frontera from "/sectores_frontera.json?url";

onMounted(() => {
  var map = L.map("mi-mapa").setView([33.505, -105], 5);

  L.tileLayer("https://tile.openstreetmap.de/{z}/{x}/{y}.png", {
    maxZoom: 18,
    attribution:
      '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
  }).addTo(map);
  d3.json("/sectores_frontera.json").then((sectores_json) => {
    console.log(sectores_json);
    var escalaColor = d3.scaleLinear()
    .domain(
      [ d3.min(sectores_json.features.map(d=>d.properties.Shape_Area)),
        d3.mean(sectores_json.features.map(d=>d.properties.Shape_Area)),
        d3.max(sectores_json.features.map(d=>d.properties.Shape_Area))
      ])
    .range(["#31081F","#6B0F1A", "#B91372"])



    L.geoJSON(sectores_json, {
      style: (feature) => ({
        color: "#fff",
        fillColor: escalaColor( feature.properties.Shape_Area),
        fillOpacity:1
      }),
      onEachFeature: (feature, layer) =>{
        layer.bindPopup(` El área es: ${feature.properties.Shape_Area}`)
      }
    }).addTo(map);
  });
});
</script>

<template>
  <div id="mi-mapa"></div>
</template>
<style scoped>
#mi-mapa {
  height: 500px;
}
</style>
