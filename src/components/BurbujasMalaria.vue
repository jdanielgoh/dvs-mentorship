<script setup>
import { onMounted, ref, watch } from "vue";
import * as d3 from "d3";

const ancho = ref(0);
const alto = ref(0);
const escala_radio = ref();

//  Elementos de seleccion de d3
const svg = ref(),
  grupo_burbujas = ref(),
  burbujas = ref(),
  burbujas_suspected = ref(),
  burbujas_RDT_positive = ref(),
  burbujas_RDT_suspect = ref();
// base de datos
const datos = ref(),
  datos_filtrados = ref(),
  anio_seleccionado = ref(2014);

// Elementos para dimensiones y fuerzas
const simulacion = ref(d3.forceSimulation());

const diccionario_color = {
  "Africa": "#8c00ff",
  "America": "#8c00ff",
  "South-East Asia": "#fcba03" ,
  "Europe":"blue",
  "Eastern Mediterranean":"#e3e4f0",
  "Western Pacific": "rgb(100,40,160)"

}

onMounted(() => {
  svg.value = d3.select("svg#burbujas");
  grupo_burbujas.value = svg.value.select("g.grupo-burbujas");

  window.addEventListener("resize", reescalando);

  d3.csv("/diagnosticos_malaria.csv").then((data) => {
    // Convertimos valores string a numéricos
    data.forEach((d) => {
      d.RDT_positive = +d.RDT_positive;
      d.RDT_suspect = +d.RDT_suspect;
      d.suspected = +d.suspected;
    });

    datos.value = data;
    // datos_filtrados será dinámica, se actualiza con el selector
    datos_filtrados.value = data.filter(
      (d) => d.Year == anio_seleccionado.value
    );
    console.log(datos_filtrados.value)

    generaEscalas();
    reiniciandoSimulacion();
    creaBurbujas();
    dibujaBurbujas();
  });
});
function reescalando() {
  generaEscalas();
  reiniciandoSimulacion();
  dibujaBurbujas();
}

/**
 * Paso 1 (se puede ejecutar varias veces):
 * dimensiones del svg y escala de los círculos
 */

function generaEscalas() {
  ancho.value = document.querySelector(
    "div.contenedor-svg-burbujas"
  ).clientWidth;
  alto.value = ancho.value;

  escala_radio.value = 0.000015 * ancho.value;
}

/**
 * Paso 2 (se puede ejecutar varias veces):
 * A la simulación le especificamos cuales son los nodos/datos
 * que usará, así como sus fuerzas.
 * Al final agregamos alpha y restart para poder
 * reciclar la función
 *
 */

function reiniciandoSimulacion() {
  simulacion.value
    .nodes(datos_filtrados.value)
    .force("x", d3.forceX().x(ancho.value / 2))
    .force("y", d3.forceY().y(alto.value / 2))
    .force("center", d3.forceCenter(ancho.value / 2, alto.value / 2))
    
    .force("charge", d3.forceManyBody())
    .on("tick", () => {
      burbujas.value.attr("transform", (d) => `translate(${d.x},${d.y})`);
      burbujas.value.selectAll(".rdt_positive")
      .attr("transform", (d) => {
        let angulo = .5 *Math.PI;
        let rad = escala_radio.value * (Math.sqrt(d.suspected) - Math.sqrt(d.RDT_positive))
        return `translate(${rad * Math.cos(angulo)},${rad * Math.sin(angulo)})`
      })
      burbujas.value.selectAll(".rdt_suspect")
      .attr("transform", (d) => {
        let angulo = .5 *Math.PI;
        let rad = escala_radio.value * (Math.sqrt(d.suspected) - Math.sqrt(d.RDT_suspect))
        return `translate(${rad * Math.cos(angulo)},${rad * Math.sin(angulo)})`
      })
    })
    .alpha(1)
    .force(
      "collide",
      d3
        .forceCollide()
        .radius((d) => 5 + escala_radio.value * Math.sqrt(d.suspected))
    )
    .restart();
}
/**
 * Paso 3 (Se ejecuta una sola vez):
 * Se crean las burbujas
 */
function creaBurbujas() {
  burbujas.value = grupo_burbujas.value
    .selectAll("burbujas")
    .data(datos_filtrados.value)
    .enter()
    .append("g");
  burbujas_suspected.value = burbujas.value.append("circle").attr("class", "suspected");
  burbujas_RDT_positive.value = burbujas.value.append("circle").attr("class", "rdt_positive");
  burbujas_RDT_suspect.value = burbujas.value.append("circle").attr("class", "rdt_suspect");

}
/**
 * Paso 4 (Se puede ejecutar varias veces):
 * Asignamos y actualizamos atributos visuales
 * como tamaños, posiciones, colores, etc
 */
function dibujaBurbujas() {
  burbujas.value.data(datos_filtrados.value);

  burbujas_suspected.value = burbujas.value
    .selectAll("circle.suspected")
    .data(d=>[d])
.transition().duration(500)
    .attr("r", (d) => escala_radio.value * Math.sqrt(d.suspected))
    .attr("fill-opacity", ".4")
    .attr("fill",d=>diccionario_color[d.Region]);


    burbujas_RDT_positive.value = burbujas.value
      .selectAll("circle.rdt_positive")
      .data(d=>[d])
      .transition().duration(500)
      .attr("r", (d) => {return escala_radio.value * Math.sqrt(d.RDT_positive)})
      .attr("fill-opacity", ".4")
      .attr("fill",d=>diccionario_color[d.Region]);

    burbujas_RDT_suspect.value = burbujas.value
      .selectAll("circle.rdt_suspect")
      .data(d=>[d])
      .transition().duration(500)
      .attr("r", (d) =>  escala_radio.value * Math.sqrt(d.RDT_suspect))
      .attr("fill-opacity", ".4")
      .attr("fill",d=>diccionario_color[d.Region]);

}

/** Una función muy poderosa de Vue que nos ayuda a
 * ver cuando una variable reactiva cambia y así
 * podemos disparar funciones
 */
watch(anio_seleccionado, () => {
  datos_filtrados.value = datos.value.filter(
    (d) => d.Year == anio_seleccionado.value
  );
  reiniciandoSimulacion();
  dibujaBurbujas();
});
</script>

<template>
  <div>
    <select name="years" id="years" v-model="anio_seleccionado">
      <option :value="year" v-for="year in d3.range(2001, 2022)" :key="year">
        {{ year }}
      </option>
    </select>
  </div>
  <div class="contenedor-svg-burbujas">
    <svg id="burbujas" :width="ancho" :height="alto">
      <g class="grupo-burbujas"></g>
    </svg>
  </div>
</template>
<style>
.contenedor-svg-burbujas {
  max-width: 700px;
  width: 100%;
  margin: auto;
}
svg {
}
</style>
