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

  // Color scale: give me a specie name, I return a color
  const color = d3.scaleOrdinal()
    .domain(["setosa", "versicolor", "virginica"])
    .range(["#440154ff", "#21908dff", "#fde725ff"])

  // Photo shown for each species (served from /public)
  const photo = {
    setosa: "/iris_setosa.png",
    versicolor: "/iris_versicolor.png",
    virginica: "/iris_virginica.png",
  }

  // Photos are stacked (top -> bottom) to the left of the "Petal_Length" axis
  const photoOrder = ["virginica", "versicolor", "setosa"]
  const photoAspect = 159 / 156 // photos are ~159x156, used to avoid side white bars

  // Here I set the list of dimension manually to control the order of axis:
  const dimensions = ["Petal_Length", "Petal_Width", "Sepal_Length", "Sepal_Width"]

  const frameT = 6    // frame thickness
  const axisLabel = 18 // room taken by the Petal_Length axis tick labels
  const gap = 12      // small visible gap between the photos and that axis' labels

  // Draw (or redraw) the chart for the container's current size
  const render = () => {
    const containerRect = container.getBoundingClientRect()

    // When navigating between slides the component can mount while still
    // hidden/animating, so the container has no size yet. Skip until it does.
    if (containerRect.width < 1 || containerRect.height < 1) return

    // Vertical margins are fixed, so the plot height (and the photo sizes that
    // depend on it) can be computed before the left margin.
    const margin = { top: 30, right: 50, bottom: 10, left: 0 }
    const height = Math.min(containerRect.height, window.innerHeight * 0.6) - margin.top - margin.bottom

    // Three photos distributed equally over the axis height; sized so their
    // frames never overlap vertically.
    const slot = height / 3
    const imgH = Math.max(1, slot - (2 * frameT + 8))
    const imgW = imgH * photoAspect

    // The left margin hosts the photos to the left of the first axis
    margin.left = imgW + gap + axisLabel + frameT + 6
    const width = containerRect.width - margin.left - margin.right

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
        .domain([0, 8]) // --> Same axis range for each group
        // --> different axis range for each group --> .domain( d3.extent(data, d => +d[name]) )
        .range([height, 0])
    }

    // Build the X scale -> it finds the best position for each Y axis
    const x = d3.scalePoint()
      .range([0, width])
      .domain(dimensions)

    // Draw the photos: always visible, to the left of the "Petal_Length" axis
    // (x = 0 here). The colored frame stays hidden until the species is hovered.
    const imgX = -(imgW + gap + axisLabel)
    const frames = {}
    photoOrder.forEach((species, i) => {
      const centerY = height * (2 * i + 1) / 6
      const imgY = centerY - imgH / 2

      const g = svg.append("g")
      // Colored frame behind the photo, revealed on hover
      frames[species] = g.append("rect")
        .attr("x", imgX - frameT)
        .attr("y", imgY - frameT)
        .attr("width", imgW + 2 * frameT)
        .attr("height", imgH + 2 * frameT)
        .attr("fill", color(species))
        .style("display", "none")
      // The photo itself
      g.append("image")
        .attr("x", imgX)
        .attr("y", imgY)
        .attr("width", imgW)
        .attr("height", imgH)
        .attr("preserveAspectRatio", "xMidYMid meet")
        .attr("href", photo[species])
        .attr("xlink:href", photo[species])
    })

    // Highlight the specie that is hovered
    const highlight = function(event, d) {
      const selected_specie = d.Species

      // first every group turns grey
      d3.selectAll(".line")
        .transition().duration(200)
        .style("stroke", "lightgrey")
        .style("opacity", "0.2")
      // Second the hovered specie takes its color
      d3.selectAll("." + selected_specie)
        .transition().duration(200)
        .style("stroke", color(selected_specie))
        .style("opacity", "1")

      // Frame the photo of the hovered specie (only one frame at a time)
      Object.values(frames).forEach(f => f.style("display", "none"))
      frames[selected_specie].style("display", null)
    }

    // Unhighlight
    const doNotHighlight = function(event, d) {
      d3.selectAll(".line")
        .transition().duration(200).delay(1000)
        .style("stroke", d => color(d.Species))
        .style("opacity", "1")

      // Remove all photo frames
      Object.values(frames).forEach(f => f.style("display", "none"))
    }

    // The path function takes a row of the csv as input, and returns x and y coordinates of the line to draw for this row.
    function path(d) {
      return d3.line()(dimensions.map(p => [x(p), y[p](d[p])]))
    }

    // Draw the lines
    svg
      .selectAll("myPath")
      .data(data)
      .join("path")
        .attr("class", d => "line " + d.Species) // 2 class for each line: 'line' and the group name
        .attr("d", path)
        .style("fill", "none")
        .style("stroke", d => color(d.Species))
        .style("opacity", 0.5)
        .on("mouseover", highlight)
        .on("mouseleave", doNotHighlight)

    // Draw the axis:
    svg.selectAll("myAxis")
      // For each dimension of the dataset I add a 'g' element:
      .data(dimensions).enter()
      .append("g")
      .attr("class", "axis")
      // I translate this element to its right position on the x axis
      .attr("transform", d => `translate(${x(d)})`)
      // And I build the axis with the call function
      .each(function(d) { d3.select(this).call(d3.axisLeft().ticks(5).scale(y[d])) })
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
