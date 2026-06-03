<script setup>
import * as d3 from 'd3'
import { sankey as d3Sankey, sankeyLinkHorizontal } from 'd3-sankey'
import { onMounted, onUnmounted, ref, nextTick } from 'vue'

const svgRef = ref(null)
const containerRef = ref(null)

let resizeObserver = null

// Build the {nodes, links} graph for a parallel-sets / Sankey diagram by
// chaining each pair of adjacent categorical columns.
function graph(data) {
  const keys = data.columns.slice(0, -1)
  let index = -1
  const nodes = []
  const nodeByKey = new d3.InternMap([], JSON.stringify)
  const indexByKey = new d3.InternMap([], JSON.stringify)
  const links = []

  for (const k of keys) {
    for (const d of data) {
      const key = [k, d[k]]
      if (nodeByKey.has(key)) continue
      const node = { name: d[k] }
      nodes.push(node)
      nodeByKey.set(key, node)
      indexByKey.set(key, ++index)
    }
  }

  for (let i = 1; i < keys.length; ++i) {
    const a = keys[i - 1]
    const b = keys[i]
    const prefix = keys.slice(0, i + 1)
    const linkByKey = new d3.InternMap([], JSON.stringify)
    for (const d of data) {
      const names = prefix.map(k => d[k])
      const value = d.value || 1
      let link = linkByKey.get(names)
      if (link) { link.value += value; continue }
      link = {
        source: indexByKey.get([a, d[a]]),
        target: indexByKey.get([b, d[b]]),
        names,
        value
      }
      links.push(link)
      linkByKey.set(names, link)
    }
  }

  return { nodes, links }
}

onMounted(async () => {
  // Wait for the next tick to ensure the container is rendered
  await nextTick()

  const container = containerRef.value

  // Parse the data once (autoType coerces the "value" column to a number)
  const data = await d3.csv("/titanic.csv", d3.autoType)

  // Color the flows by survival; everything else (Survived) falls back to grey
  const color = d3.scaleOrdinal(["Perished"], ["#da4f81"]).unknown("#ccc")

  // Draw (or redraw) the diagram for the container's current size
  const render = () => {
    // Use clientWidth/Height (layout pixels), not getBoundingClientRect():
    // Slidev CSS-scales the slide, which would otherwise distort the size.
    const width = container.clientWidth
    const height = container.clientHeight

    // Skip while hidden/animating (e.g. when navigating between slides)
    if (width < 1 || height < 1) return

    const sankey = d3Sankey()
      .nodeSort(null)
      .linkSort(null)
      .nodeWidth(4)
      .nodePadding(20)
      .extent([[0, 5], [width, height - 5]])

    const svg = d3.select(svgRef.value)
      .attr("viewBox", `0 0 ${width} ${height}`)
      .attr("width", width)
      .attr("height", height)
      .attr("style", "max-width: 100%; height: auto;")

    // Clear any previous render before drawing again
    svg.selectAll("*").remove()

    const g = graph(data)
    const { nodes, links } = sankey({
      nodes: g.nodes.map(d => Object.create(d)),
      links: g.links.map(d => Object.create(d))
    })

    // Nodes
    svg.append("g")
      .selectAll("rect")
      .data(nodes)
      .join("rect")
        .attr("x", d => d.x0)
        .attr("y", d => d.y0)
        .attr("height", d => d.y1 - d.y0)
        .attr("width", d => d.x1 - d.x0)
      .append("title")
        .text(d => `${d.name}\n${d.value.toLocaleString()}`)

    // Links
    svg.append("g")
        .attr("fill", "none")
      .selectAll("g")
      .data(links)
      .join("path")
        .attr("d", sankeyLinkHorizontal())
        .attr("stroke", d => color(d.names[0]))
        .attr("stroke-width", d => d.width)
        .style("mix-blend-mode", "multiply")
      .append("title")
        .text(d => `${d.names.join(" → ")}\n${d.value.toLocaleString()}`)

    // Labels
    svg.append("g")
        .style("font", "10px sans-serif")
      .selectAll("text")
      .data(nodes)
      .join("text")
        .attr("x", d => d.x0 < width / 2 ? d.x1 + 6 : d.x0 - 6)
        .attr("y", d => (d.y1 + d.y0) / 2)
        .attr("dy", "0.35em")
        .attr("text-anchor", d => d.x0 < width / 2 ? "start" : "end")
        .text(d => d.name)
      .append("tspan")
        .attr("fill-opacity", 0.7)
        .text(d => ` ${d.value.toLocaleString()}`)
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
