<script setup>
import { onMounted, ref } from "vue";
import * as d3 from "d3";
const margenes = ref({
  arriba: 20,
  abajo: 20,
  izquierda: 50,
  derecha: 20,
});
const ancho = ref(0);
const alto = ref(0);

//  Elementos de seleccion de d3
const svg = ref(), grupo_poligonos = ref(), paths_sectores= ref();
// base de datos
const geojson_sectores = ref();
// Elementos para dimensiones y escalas de mapa
const proyeccion = ref(d3.geoMercator()), geopath = ref(d3.geoPath()), escalaColor = ref(d3.scaleLinear())
onMounted(() => {
  svg.value = d3.select("svg#mapa");
  grupo_poligonos.value = svg.value.select("g.grupo-poligonos");


  window.addEventListener("resize", reescalando);
  d3.json("/sectores_frontera.json").then((data) => {

    geojson_sectores.value = data; 
    generaEscalas()
    creaPoligonos()
    dibujaPoligonos()

  });
});
function reescalando() {
  generaEscalas()
  dibujaPoligonos()
}

function generaEscalas() {
  ancho.value =
    document.querySelector("div#app").clientWidth
  
  alto.value = ancho.value * 0.5
  
  // La proyección transforma de un sistema de coordenadas a otro 
  proyeccion.value
    .scale(1.5 * ancho.value)
    .center([-107, 34.5])
    .translate([.5*ancho.value, .5* alto.value])
  
  // Geopath sirve para pintar esa proyección  
  geopath.value.projection(proyeccion.value)
  // Escala para colorear mapa
  escalaColor.value
    .domain(
      [ d3.min(geojson_sectores.value.features.map(d=>d.properties.Shape_Area)),
        d3.mean(geojson_sectores.value.features.map(d=>d.properties.Shape_Area)),
        d3.max(geojson_sectores.value.features.map(d=>d.properties.Shape_Area))
      ])
    .range(["#31081F","#6B0F1A", "#B91372"])

}

function creaPoligonos() {
  paths_sectores.value = grupo_poligonos.value.selectAll("paths")
    .data(geojson_sectores.value.features)
    .enter()
    .append("path")
}
function dibujaPoligonos() {
  paths_sectores.value
    .attr("d", geopath.value)
    .attr("fill",d=>escalaColor.value(d.properties.Shape_Area))
    .attr("stroke", "#fff")
}

</script>

<template>
  <svg
    id="mapa"
    :width="ancho + margenes.derecha + margenes.izquierda"
    :height="alto + margenes.arriba + margenes.abajo"
  >
    <g
      class="grupo-poligonos"
      :transform="`translate(${margenes.izquierda}, ${margenes.arriba})`"
    ></g>
  </svg>
</template>


