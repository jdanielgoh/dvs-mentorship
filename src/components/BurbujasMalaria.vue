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
const svg = ref(),
  grupo_burbujas = ref(),
  burbujas = ref();
// base de datos
const datos = ref(),
  datos_filtrados = ref();
// Elementos para dimensiones y fuerzas
const simulacion = ref(
  d3.forceSimulation().force("charge", d3.forceManyBody())
);

onMounted(() => {
  svg.value = d3.select("svg#burbujas");
  grupo_burbujas.value = svg.value.select("g.grupo-burbujas");

  window.addEventListener("resize", reescalando);
  d3.csv("/diagnosticos_malaria.csv").then((data) => {
    data.forEach((d) => {
      d.RDT_positive = +d.RDT_positive;
      d.RDT_suspect = +d.RDT_suspect;
      d.suspected = +d.suspected;
    });
    datos.value = data;
    datos_filtrados.value = data.filter((d) => d.Year == 2020);
    simulacion.value.nodes(datos_filtrados.value);

    generaEscalas();
    creaBurbujas();
    dibujaBurbujas();
  });
});
function reescalando() {}
function generaEscalas() {
  ancho.value = document.querySelector("div#app").clientWidth;
  alto.value = ancho.value * 0.5;
  simulacion.value.force("center", d3.forceCenter(ancho.value / 2, alto.value / 2))
    .on("tick",ticked);
}

function creaBurbujas() {
  burbujas.value = grupo_burbujas.value
    .selectAll("burbujas")
    .data(datos_filtrados.value)
    .enter()
    .append("circle");
}
function dibujaBurbujas() {
  burbujas.value.attr("r", 10);
}
function ticked(){
  burbujas.value.attr("transform",d=>`translate(${d.x},${d.y})`)
}
</script>

<template>
  <svg
    id="burbujas"
    :width="ancho + margenes.derecha + margenes.izquierda"
    :height="alto + margenes.arriba + margenes.abajo"
  >
    <g
      class="grupo-burbujas"
      :transform="`translate(${margenes.izquierda}, ${margenes.arriba})`"
    ></g>
  </svg>
</template>
