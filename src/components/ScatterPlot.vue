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
const alto = ref(600 - margenes.value.arriba - margenes.value.abajo);

const svg = ref(),
  eje_x = ref(),
  eje_y = ref();
const grupo_burbujas = ref();

const escalaX = ref(),
  escalaY = ref(),
  escalaColor = ref(),
  datos = ref();
onMounted(() => {
  svg.value = d3.select("svg#viz");
  grupo_burbujas.value = svg.value.select("g.grupo-burbujas");
  eje_x.value = svg.value.select("g.eje-x");
  eje_y.value = svg.value.select("g.eje-y");

  window.addEventListener("resize", reescalando);
  d3.csv("/CaltechExoplanetArchive.csv").then((data) => {
    data.forEach((d) => {
      d.disc_year = +d.disc_year;
      d.pl_masse = +d.pl_masse;
      d.sy_dist = +d.sy_dist
    });
    datos.value = data.filter(d=>d.pl_masse!=0);
    console.log(datos.value.length)
    generaEscalas();
    creaBurbujas();
    dibujaBurbujas();
  });
});
function reescalando() {
  generaEscalas();
  dibujaBurbujas();
}

function generaEscalas() {
  ancho.value =
    document.querySelector("div#app").clientWidth -
    margenes.value.derecha -
    margenes.value.izquierda;
  escalaY.value = d3
    .scaleLinear()
    .domain(d3.extent(datos.value.map((d) => d.disc_year)))
    .range([0, alto.value]);
  escalaX.value = d3
    .scaleLinear()
    .domain(d3.extent(datos.value.map((d) => d.sy_dist)))
    .range([0, ancho.value]);

  escalaColor.value = d3
    .scaleOrdinal()
    .domain([...new Set(datos.value.map((d) => d.discoverymethod))])
    .range([
      "#1f77b4",
      "#ff7f0e",
      "#2ca02c",
      "#d62728",
      "#9467bd",
      "#8c564b",
      "#e377c2",
      "#7f7f7f",
      "#bcbd22",
      "#17becf",
      "red",
    ]);

  eje_x.value.call(d3.axisBottom(escalaX.value).ticks(10));
  eje_y.value
    .call(d3.axisLeft(escalaY.value))
    .selectAll("line")
    .attr("x1", ancho.value)
    .style("stroke", "gray")
    .style("stroke-dasharray", "2 2");
}

function creaBurbujas() {
  grupo_burbujas.value
    .selectAll("circulos")
    .data(datos.value)
    .enter()
    .append("circle")
    .on("mouseover", mouseOver)
    .on("mouseout", mouseOut);
}
function dibujaBurbujas() {
  grupo_burbujas.value
    .selectAll("circle")
    .attr("cy", (d) => escalaY.value(d.disc_year))
    .attr("cx", (d) => escalaX.value(d.sy_dist))
    .attr("r", (d) => Math.pow(d.pl_masse, 0.5) / 10)
    .style("fill", (d) => escalaColor.value(d.discoverymethod))
    .style("fill-opacity", ".5");
}
function mouseOver(e, d) {
  d3.select(this)
    .transition()
    .duration(200)
    .attr("r", (d) => Math.pow(d.pl_masse, 0.5) / 5)
    .style("fill-opacity", "1");
}
function mouseOut(e, d) {
  d3.select(this)
    .transition()
    .duration(200)
    .attr("r", (d) => Math.pow(d.pl_masse, 0.5) / 10)
    .style("fill-opacity", ".5");
}
</script>

<template>
  <svg
    id="viz"
    :width="ancho + margenes.derecha + margenes.izquierda"
    :height="alto + margenes.arriba + margenes.abajo"
  >
    <g
      class="eje-x"
      :transform="`translate(${margenes.izquierda}, ${margenes.arriba + alto})`"
    ></g>
    <g
      class="eje-y"
      :transform="`translate(${margenes.izquierda}, ${margenes.arriba})`"
    ></g>

    <g
      class="grupo-burbujas"
      :transform="`translate(${margenes.izquierda}, ${margenes.arriba})`"
    ></g>
  </svg>
</template>

<style scoped>
svg#viz {
}
</style>
