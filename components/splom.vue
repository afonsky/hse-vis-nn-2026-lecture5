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

  // Parse the data once (autoType coerces the measurements; "NaN" -> NaN)
  const data = await d3.csv("/penguins.csv", d3.autoType)

  // Keep only the numeric columns for the matrix
  const columns = data.columns.filter(d => typeof data[0][d] === "number")

  // Define the color scale.
  const color = d3.scaleOrdinal()
    .domain(data.map(d => d.species))
    .range(d3.schemeCategory10)

  // Draw (or redraw) the matrix for the container's current size
  const render = () => {
    // Use clientWidth/Height (layout pixels), not getBoundingClientRect():
    // Slidev CSS-scales the slide, which would otherwise distort the size.
    // The matrix is square, so size it to the smaller dimension.
    const width = Math.min(container.clientWidth, container.clientHeight)
    const height = width

    // Skip while hidden/animating (e.g. when navigating between slides)
    if (width < 1) return

    const padding = 28
    const size = (width - (columns.length + 1) * padding) / columns.length + padding

    // Horizontal scales (one per row) and companion vertical scales (one per column).
    const x = columns.map(c => d3.scaleLinear()
      .domain(d3.extent(data, d => d[c]))
      .rangeRound([padding / 2, size - padding / 2]))
    const y = x.map(x => x.copy().range([size - padding / 2, padding / 2]))

    // Horizontal axis (applied separately for each column).
    const axisx = d3.axisBottom().ticks(6).tickSize(size * columns.length)
    const xAxis = g => g.selectAll("g").data(x).join("g")
      .attr("transform", (d, i) => `translate(${i * size},0)`)
      .each(function (d) { return d3.select(this).call(axisx.scale(d)) })
      .call(g => g.select(".domain").remove())
      .call(g => g.selectAll(".tick line").attr("stroke", "#ddd"))

    // Vertical axis (applied separately for each row).
    const axisy = d3.axisLeft().ticks(6).tickSize(-size * columns.length)
    const yAxis = g => g.selectAll("g").data(y).join("g")
      .attr("transform", (d, i) => `translate(0,${i * size})`)
      .each(function (d) { return d3.select(this).call(axisy.scale(d)) })
      .call(g => g.select(".domain").remove())
      .call(g => g.selectAll(".tick line").attr("stroke", "#ddd"))

    const svg = d3.select(svgRef.value)
      .attr("width", width)
      .attr("height", height)
      .attr("viewBox", [-padding, 0, width, height])
      .attr("style", "max-width: 100%; height: auto;")

    // Clear any previous render before drawing again
    svg.selectAll("*").remove()

    svg.append("style")
      .text(`circle.hidden { fill: #000; fill-opacity: 1; r: 1px; }`)

    svg.append("g").call(xAxis)
    svg.append("g").call(yAxis)

    const cell = svg.append("g")
      .selectAll("g")
      .data(d3.cross(d3.range(columns.length), d3.range(columns.length)))
      .join("g")
        .attr("transform", ([i, j]) => `translate(${i * size},${j * size})`)

    cell.append("rect")
      .attr("fill", "none")
      .attr("stroke", "#aaa")
      .attr("x", padding / 2 + 0.5)
      .attr("y", padding / 2 + 0.5)
      .attr("width", size - padding)
      .attr("height", size - padding)

    cell.each(function ([i, j]) {
      d3.select(this).selectAll("circle")
        .data(data.filter(d => !isNaN(d[columns[i]]) && !isNaN(d[columns[j]])))
        .join("circle")
          .attr("cx", d => x[i](d[columns[i]]))
          .attr("cy", d => y[j](d[columns[j]]))
    })

    const circle = cell.selectAll("circle")
      .attr("r", 3.5)
      .attr("fill-opacity", 0.7)
      .attr("fill", d => color(d.species))

    // Brushing: highlight points within the selection across the whole matrix.
    const brush = d3.brush()
      .extent([[padding / 2, padding / 2], [size - padding / 2, size - padding / 2]])
      .on("start", brushstarted)
      .on("brush", brushed)
      .on("end", brushended)

    cell.call(brush)

    let brushCell

    // Clear the previously-active brush, if any.
    function brushstarted() {
      if (brushCell !== this) {
        d3.select(brushCell).call(brush.move, null)
        brushCell = this
      }
    }

    // Highlight the selected circles.
    function brushed({ selection }, [i, j]) {
      if (selection) {
        const [[x0, y0], [x1, y1]] = selection
        circle.classed("hidden",
          d => x0 > x[i](d[columns[i]])
            || x1 < x[i](d[columns[i]])
            || y0 > y[j](d[columns[j]])
            || y1 < y[j](d[columns[j]]))
      }
    }

    // If the brush is empty, select all circles.
    function brushended({ selection }) {
      if (selection) return
      circle.classed("hidden", false)
    }

    // Column labels on the diagonal
    svg.append("g")
        .style("font", "bold 10px sans-serif")
        .style("pointer-events", "none")
      .selectAll("text")
      .data(columns)
      .join("text")
        .attr("transform", (d, i) => `translate(${i * size},${i * size})`)
        .attr("x", padding)
        .attr("y", padding)
        .attr("dy", ".71em")
        .text(d => d)
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
  <div ref="containerRef" class="w-full h-[460px] flex justify-center items-center overflow-hidden">
    <svg ref="svgRef"></svg>
  </div>
</template>
