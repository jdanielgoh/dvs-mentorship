<script setup>
import { onMounted, ref, watch } from "vue";
import * as d3 from "d3";

const ancho = ref(0);
const alto = ref(0);
const escala_radio = ref();

const tooltip_visible = ref(false),
  posicion_x = ref(0),
  posicion_y = ref(0),
  html_tooltip = ref("");

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

const diccionario_color = ref({
  Africa: "#004f5f",
  America: "#766aaf",
  "South-East Asia": "#38c7a6",
  Europe: "#36e9fe",
  "Eastern Mediterranean": "#f9f871",
  "Western Pacific": "rgb(255,0,0)",
});

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
      burbujas.value.selectAll(".rdt_positive").attr("transform", (d) => {
        let angulo = 0.5 * Math.PI;
        let rad =
          escala_radio.value *
          (Math.sqrt(d.suspected) - Math.sqrt(d.RDT_positive));
        return `translate(${rad * Math.cos(angulo)},${rad * Math.sin(angulo)})`;
      });
      burbujas.value.selectAll(".rdt_suspect").attr("transform", (d) => {
        let angulo = 0.5 * Math.PI;
        let rad =
          escala_radio.value *
          (Math.sqrt(d.suspected) - Math.sqrt(d.RDT_suspect));
        return `translate(${rad * Math.cos(angulo)},${rad * Math.sin(angulo)})`;
      });

      burbujas.value
      .selectAll("text.label")
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
    .append("g")
    .on("mousemove", (e, d) => {
      tooltip_visible.value = true;
      posicion_x.value = e.layerX - 100 + "px";
      posicion_y.value = e.layerY - 130 + "px";
      html_tooltip.value = `
      <p>RDT positive: <b>${d.RDT_positive.toLocaleString("en-US")} </b></p>
      `;
    })
    .on("mouseout", (e, d) => {
      tooltip_visible.value = false;
    });
  burbujas_suspected.value = burbujas.value
    .append("circle")
    .attr("class", "suspected");
  burbujas_RDT_positive.value = burbujas.value
    .append("circle")
    .attr("class", "rdt_positive");
  burbujas_RDT_suspect.value = burbujas.value
    .append("circle")
    .attr("class", "rdt_suspect");

  /*add text*/
  /*burbujas.value
    .append("text")
    .attr("class", "label")
    .text((d) => d.Region) // Assuming 'Region' is the field you want to label
    .attr("text-anchor", "middle")
    .attr("dy", ".35em"); // Vertically center text
    */
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
    .data((d) => [d])
    .transition()
    .duration(500)
    .attr("r", (d) => escala_radio.value * Math.sqrt(d.suspected))
    .attr("fill-opacity", ".4")
    .attr("fill", (d) => diccionario_color.value[d.Region]);

  burbujas_RDT_positive.value = burbujas.value
    .selectAll("circle.rdt_positive")
    .data((d) => [d])
    .transition()
    .duration(500)
    .attr("r", (d) => {
      return escala_radio.value * Math.sqrt(d.RDT_positive);
    })
    .attr("fill-opacity", ".4")
    .attr("fill", (d) => diccionario_color.value[d.Region]);

  burbujas_RDT_suspect.value = burbujas.value
    .selectAll("circle.rdt_suspect")
    .data((d) => [d])
    .transition()
    .duration(500)
    .attr("r", (d) => escala_radio.value * Math.sqrt(d.RDT_suspect))
    .attr("fill-opacity", ".4")
    .attr("fill", (d) => diccionario_color.value[d.Region]);

  /*add text*/
  // Update text labels
  /*burbujas.value
    .selectAll("text.label")
    .data((d) => [d])
    .transition()
    .duration(500)
   
    .style(
      "font-size",
      (d) =>
        Math.min(2 * escala_radio.value * Math.sqrt(d.suspected), 16) + "px"
    ) // Adjust font size based on bubble size
    .text((d) => d.Region); // Update text if necessary
    */
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
    <div
      class="tooltip"
      :style="{
        display: tooltip_visible ? 'block' : 'none',
        top: posicion_y,
        left: posicion_x,
      }"
      v-html="html_tooltip"
    ></div>
    <svg id="burbujas" :width="ancho" :height="alto">
      <g class="grupo-burbujas"></g>
    </svg>
  </div>
  <div class="nomenclaturas">
    <p>
      <span v-for="(clave,i) in  Object.keys(diccionario_color)" :key="i" class="nomenclatura-region"> 
        <span class="nomenclatura-circulo" :style="{background: diccionario_color[clave]}"></span>
        {{ clave }}
      </span>
    </p>

  </div>
</template>
<style>
.contenedor-svg-burbujas {
  max-width: 700px;
  width: 100%;
  margin: auto;
  position: relative;
}
.tooltip {
  font-family: "Roboto";
  height: 100px;
  background: rgba(0, 0, 0, 0.8);
  position: absolute;
  z-index: 2;
  width: 200px;
  padding: 8px;
  color: white;
  border-radius: 8px;
}
.nomenclatura-circulo{
  width: 16px;
  height: 16px;
  display: inline-block;
  border-radius: 50%;
  transform: translateY(2px)
}
.nomenclatura-region{
  margin: 0 8px;
  display: block;
}
svg {
  position: relative;
  z-index: 1;
}
</style>
