<script setup>
import * as d3 from 'd3'
import { onMounted, onUnmounted, ref, nextTick } from 'vue'

const svgRef = ref(null)
const containerRef = ref(null)

let resizeObserver = null

onMounted(async () => {
  // Wait for the next tick to ensure the container is rendered
  await nextTick()

  const container = containerRef.value

  // Parse the Data once; redraws reuse it
  const data = await d3.csv("https://raw.githubusercontent.com/holtzy/D3-graph-gallery/master/DATA/iris.csv")

  // Extract the list of dimensions we want to keep in the plot. Here I keep all except the column called Species
  const dimensions = Object.keys(data[0]).filter(d => d != "Species")

  // Draw (or redraw) the chart for the container's current size
  const render = () => {
    const containerRect = container.getBoundingClientRect()

    // When navigating between slides the component can mount while still
    // hidden/animating, so the container has no size yet. Skip until it does.
    if (containerRect.width < 1 || containerRect.height < 1) return

    // Set the dimensions and margins of the graph
    const margin = { top: 30, right: 10, bottom: 10, left: 0 }
    const width = containerRect.width - margin.left - margin.right
    const height = Math.min(containerRect.height, window.innerHeight * 0.6) - margin.top - margin.bottom

    // Clear any previous render before drawing again
    const svgEl = d3.select(svgRef.value)
    svgEl.selectAll("*").remove()

    // Append a group for the margins
    const svg = svgEl
      .attr("width", width + margin.left + margin.right)
      .attr("height", height + margin.top + margin.bottom)
      .attr("viewBox", `0 0 ${width + margin.left + margin.right} ${height + margin.top + margin.bottom}`)
      .append("g")
        .attr("transform", `translate(${margin.left},${margin.top})`)

    // For each dimension, I build a linear scale. I store all in a y object
    const y = {}
    for (const name of dimensions) {
      y[name] = d3.scaleLinear()
        .domain(d3.extent(data, d => +d[name]))
        .range([height, 0])
    }

    // Build the X scale -> it finds the best position for each Y axis
    const x = d3.scalePoint()
      .range([0, width])
      .padding(1)
      .domain(dimensions)

    // The path function takes a row of the csv as input, and returns x and y coordinates of the line to draw for this row.
    function path(d) {
      return d3.line()(dimensions.map(p => [x(p), y[p](d[p])]))
    }

    // Draw the lines
    svg
      .selectAll("myPath")
      .data(data)
      .join("path")
      .attr("d", path)
      .style("fill", "none")
      .style("stroke", "#69b3a2")
      .style("opacity", 0.5)

    // Draw the axis:
    svg.selectAll("myAxis")
      // For each dimension of the dataset I add a 'g' element:
      .data(dimensions).enter()
      .append("g")
      // I translate this element to its right position on the x axis
      .attr("transform", d => `translate(${x(d)})`)
      // And I build the axis with the call function
      .each(function(d) { d3.select(this).call(d3.axisLeft().scale(y[d])) })
      // Add axis title
      .append("text")
        .style("text-anchor", "middle")
        .attr("y", -9)
        .text(d => d)
        .style("fill", "black")
  }

  // ResizeObserver fires once the container gets a real size (after the slide
  // transition finishes) and again on any resize, so the chart always renders.
  resizeObserver = new ResizeObserver(() => render())
  resizeObserver.observe(container)
})

onUnmounted(() => {
  if (resizeObserver) resizeObserver.disconnect()
})
</script>

<template>
  <div ref="containerRef" class="w-full h-80 flex justify-center overflow-hidden">
    <svg ref="svgRef" class="w-full h-full" preserveAspectRatio="xMidYMid meet"></svg>
  </div>
</template>
