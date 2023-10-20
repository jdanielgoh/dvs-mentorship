<script setup>
import {onMounted} from "vue"
import * as d3 from "d3"


onMounted(()=>{
  var ancho = document.querySelector("div#app").clientWidth
  var alto = 600
  var svg = d3.select("svg#viz")
  
  svg.attr("width", ancho)
    .attr("height", 600)

  d3.csv("/CaltechExoplanetArchive.csv")
    .then((datos)=>{
      datos.forEach((d)=>{
        d.disc_year = +d.disc_year
        d.pl_masse= +d.pl_masse
      })
      console.log(datos)
      var escalaY = d3.scaleLinear()
        .domain(d3.extent(datos.map(d=>d.disc_year)))
        .range([0, alto])
      
      var escalaX = d3.scaleLinear()
        .domain(d3.extent(datos.map(d=>d.sy_dist)))
        .range([0, ancho])
      
      console.log(escalaY(1989))

      svg.selectAll("circulos")
        .data(datos)
        .enter()
        .append("circle")
        .attr("cy", d=>escalaY(d.disc_year))
        .attr("cx", d=>escalaX(d.sy_dist))
        .attr("r", d=> Math.pow(d.pl_masse, .5)/ 10)
        .style("fill", "red")
        .style("fill-opacity", ".5")

    })

})
</script>

<template>
<svg id="viz">

</svg>
</template>

<style scoped>
svg#viz{
}
</style>
