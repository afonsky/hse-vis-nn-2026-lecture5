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

  // Parse the data once (autoType coerces "year" and the numeric columns)
  const data = await d3.csv(
    "https://raw.githubusercontent.com/holtzy/data_to_viz/master/Example_dataset/5_OneCatSevNumOrdered_wide.csv",
    d3.autoType
  )

  // List of groups = header of the csv files (everything except "year")
  const keys = data.columns.slice(1)

  // color palette
  const color = d3.scaleOrdinal().domain(keys).range(d3.schemeDark2)

  // Draw (or redraw) the chart for the container's current size
  const render = () => {
    // Use clientWidth/Height (layout pixels), not getBoundingClientRect():
    // Slidev CSS-scales the slide, which would otherwise distort the size.
    const margin = { top: 20, right: 30, bottom: 0, left: 10 }
    const width = container.clientWidth - margin.left - margin.right
    const height = container.clientHeight - margin.top - margin.bottom

    // Skip while hidden/animating (e.g. when navigating between slides)
    if (width < 1 || height < 1) return

    // Clear any previous render before drawing again
    const svgEl = d3.select(svgRef.value)
    svgEl.selectAll("*").remove()

    const svg = svgEl
      .attr("width", width + margin.left + margin.right)
      .attr("height", height + margin.top + margin.bottom)
      .attr("viewBox", `0 0 ${width + margin.left + margin.right} ${height + margin.top + margin.bottom}`)
      .append("g")
        .attr("transform", `translate(${margin.left},${margin.top})`)

    // Add X axis
    const x = d3.scaleLinear()
      .domain(d3.extent(data, d => d.year))
      .range([0, width])
    svg.append("g")
      .attr("transform", `translate(0,${height * 0.8})`)
      .call(d3.axisBottom(x).tickSize(-height * 0.7).tickValues([1900, 1925, 1975, 2000]).tickFormat(d3.format("d")))
      .select(".domain").remove()
    // Customization
    svg.selectAll(".tick line").attr("stroke", "#b8b8b8")

    // Add X axis label:
    svg.append("text")
      .attr("text-anchor", "end")
      .attr("x", width)
      .attr("y", height - 30)
      .text("Время (год)")

    // Add Y axis
    const y = d3.scaleLinear()
      .domain([-100000, 100000])
      .range([height, 0])

    // stack the data
    const stackedData = d3.stack()
      .offset(d3.stackOffsetSilhouette)
      .keys(keys)
      (data)

    // create a tooltip
    const Tooltip = svg.append("text")
      .attr("x", 0)
      .attr("y", 0)
      .style("opacity", 0)
      .style("font-size", 17)

    // Three functions that change the tooltip when user hover / move / leave a cell.
    // d3 v7 passes (event, datum) to handlers; the datum is the stacked series.
    const mouseover = function (event, d) {
      Tooltip.style("opacity", 1)
      d3.selectAll(".myArea").style("opacity", 0.2)
      d3.select(this).style("stroke", "black").style("opacity", 1)
    }
    const mousemove = function (event, d) {
      Tooltip.text(d.key)
    }
    const mouseleave = function (event, d) {
      Tooltip.style("opacity", 0)
      d3.selectAll(".myArea").style("opacity", 1).style("stroke", "none")
    }

    // Area generator
    const area = d3.area()
      .x(d => x(d.data.year))
      .y0(d => y(d[0]))
      .y1(d => y(d[1]))

    // Show the areas
    svg.selectAll("mylayers")
      .data(stackedData)
      .enter()
      .append("path")
        .attr("class", "myArea")
        .style("fill", d => color(d.key))
        .attr("d", area)
        .on("mouseover", mouseover)
        .on("mousemove", mousemove)
        .on("mouseleave", mouseleave)
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
  <div ref="containerRef" class="w-full h-[400px] flex justify-center overflow-hidden">
    <svg ref="svgRef" class="w-full h-full"></svg>
  </div>
</template>
