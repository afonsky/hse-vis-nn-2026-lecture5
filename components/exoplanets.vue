<script setup>
import * as d3 from 'd3'
import { onMounted, onUnmounted, ref, nextTick } from 'vue'

const containerRef = ref(null)
let resizeObserver = null

// Minimal renderQueue (was an external lib): draws data in animation-frame
// chunks so the canvas stays responsive and brushing can interrupt a render.
function renderQueue(func) {
  let _queue = []
  let _rate = 1000
  let _invalidate = () => {}
  let _clear = () => {}
  const frame = window.requestAnimationFrame || (cb => setTimeout(cb, 17))

  function rq(data) {
    if (data) rq.data(data)
    _invalidate()
    _clear()
    rq.render()
  }
  rq.render = function () {
    let valid = true
    _invalidate = rq.invalidate = () => { valid = false }
    function doFrame() {
      if (!valid) return
      _queue.splice(0, _rate).forEach(func)
      if (_queue.length) frame(doFrame)
    }
    doFrame()
  }
  rq.data = function (data) { _invalidate(); _queue = data.slice(0); return rq }
  rq.rate = function (value) { if (!arguments.length) return _rate; _rate = value; return rq }
  rq.remaining = () => _queue.length
  rq.invalidate = _invalidate
  rq.clear = function (func) { if (!arguments.length) return _clear; _clear = func; return rq }
  return rq
}

function d3_functor(v) {
  return typeof v === 'function' ? v : function () { return v }
}

onMounted(async () => {
  // Wait for the next tick to ensure the container is rendered
  await nextTick()

  const containerEl = containerRef.value

  const margin = { top: 95, right: 60, bottom: 20, left: 120 }
  const height = 360 - margin.top - margin.bottom
  const innerHeight = height - 2
  const devicePixelRatio = window.devicePixelRatio || 1

  const color = d3.scaleOrdinal()
    .domain(["Radial Velocity", "Imaging", "Eclipse Timing Variations", "Astrometry", "Microlensing", "Orbital Brightness Modulation", "Pulsar Timing", "Pulsation Timing Variations", "Transit", "Transit Timing Variations"])
    .range(["#DB7F85", "#50AB84", "#4C6C86", "#C47DCB", "#B59248", "#DD6CA7", "#E15E5A", "#5DA5B3", "#725D82", "#54AF52", "#954D56", "#8C92E8", "#D8597D", "#AB9C27", "#D67D4B", "#D58323", "#BA89AD", "#357468", "#8F86C2", "#7D9E33", "#517C3F", "#9D5130", "#5E9ACF", "#776327", "#944F7E"])

  const types = {
    "Number": {
      key: "Number",
      coerce: d => +d,
      extent: d3.extent,
      within: (d, extent, dim) => extent[0] <= dim.scale(d) && dim.scale(d) <= extent[1],
      defaultScale: d3.scaleLinear().range([innerHeight, 0])
    },
    "String": {
      key: "String",
      coerce: String,
      extent: data => data.sort(),
      within: (d, extent, dim) => extent[0] <= dim.scale(d) && dim.scale(d) <= extent[1],
      defaultScale: d3.scalePoint().range([0, innerHeight])
    },
    "Date": {
      key: "Date",
      coerce: d => new Date(d),
      extent: d3.extent,
      within: (d, extent, dim) => extent[0] <= dim.scale(d) && dim.scale(d) <= extent[1],
      defaultScale: d3.scaleTime().range([innerHeight, 0])
    }
  }

  const dimensions = [
    {
      key: "pl_discmethod",
      description: "Discovery Method",
      type: types["String"],
      axis: d3.axisLeft().tickFormat(d => discMethodShort[d] || d)
    },
    { key: "pl_letter", description: "Planet Letter", type: types["String"] },
    { key: "pl_pnum", description: "Number of Planets in System", type: types["Number"] },
    { key: "pl_orbper", type: types["Number"], description: "Planet Orbital Period", scale: d3.scaleLog().range([innerHeight, 0]) },
    { key: "pl_orbsmax", type: types["Number"], description: "Planet Semi-Major Axis", scale: d3.scaleLog().range([innerHeight, 0]) },
    { key: "pl_orbeccen", description: "Planet Eccentricity", type: types["Number"] },
    { key: "pl_orbincl", description: "Planet Inclination", type: types["Number"] },
    { key: "pl_bmassj", description: "Mass in Jupiters", type: types["Number"] },
    { key: "pl_rade", description: "Planet Radius in Earth Radii", type: types["Number"] },
    { key: "pl_eqt", description: "Planet Equilibrium Temperature (K)", type: types["Number"] },
    { key: "pl_imppar", description: "Impact Parameter", type: types["Number"] },
    { key: "pl_trandep", description: "Transit Depth (%)", type: types["Number"] },
    { key: "pl_trandur", description: "Transit Duration (days)", type: types["Number"] },
    { key: "pl_ratror", description: "Planet-Star Radius Ratio", type: types["Number"] },
    {
      key: "pl_locale",
      type: types["String"],
      axis: d3.axisLeft().tickFormat((d, i) => d == "Multiple Locales" ? "Multiple" : d)
    },
    { key: "pl_disc", description: "Year of Discovery", type: types["Date"] },
    {
      key: "pl_facility",
      description: "Discovery Facility",
      type: types["String"],
      domain: ["Kepler", "La Silla Observatory", "K2", "W. M. Keck Observatory", "SuperWASP", "Multiple Observatories", "HATNet", "Haute-Provence Observatory", "Anglo-Australian Telescope", "OGLE", "Lick Observatory", "HATSouth", "CoRoT", "McDonald Observatory", "Okayama Astrophysical Observatory", "MOA", "Bohyunsan Optical Astronomical Observatory", "Las Campanas Observatory", "SuperWASP-South", "Roque de los Muchachos Observatory", "Paranal Observatory", "Gemini Observatory", "KELT", "Subaru Telescope", "Thueringer Landessternwarte Tautenburg", "XO", "Multiple Facilities", "Hubble Space Telescope", "Fred Lawrence Whipple Observatory", "TrES", "kepler", "KELT-South", "Spitzer Space Telescope", "Arecibo Observatory", "United Kingdom Infrared Telescope", "Large Binocular Telescope Observatory", "Xinglong Station", "Cerro Tololo Inter-American Observatory", "Palomar Observatory", "SuperWASP-North", "Qatar", "Teide Observatory", "European Southern Observatory", "Leoncito Astronomical Complex", "Infrared Survey Facility", "KMTNet", "Parkes Observatory", "Apache Point Observatory", "Oak Ridge Observatory", "MEarth Project", "Yunnan Astronomical Observatory", "Kitt Peak National Observatory"],
      // 52 facilities would stack into an unreadable jumble; hide the tick text
      // (the axis is still brushable) and keep only the title.
      axis: d3.axisRight().tickFormat(() => "")
    }
  ]

  // Shorten the few long discovery-method labels so the left margin can be tight
  const discMethodShort = {
    "Orbital Brightness Modulation": "Orbital Bright. Mod.",
    "Pulsation Timing Variations": "Pulsation Timing Var.",
    "Transit Timing Variations": "Transit Timing Var.",
    "Eclipse Timing Variations": "Eclipse Timing Var."
  }

  const yAxis = d3.axisLeft()

  // Parse the data once (d3 v7 returns a promise; served from /public)
  const data = await d3.csv("/planets.csv")

  data.forEach(d => {
    dimensions.forEach(p => {
      d[p.key] = !d[p.key] ? null : p.type.coerce(d[p.key])
    })
    // truncate long text strings
    for (const key in d) {
      if (d[key] && d[key].length > 35) d[key] = d[key].slice(0, 36)
    }
  })

  // Type/dimension default setting (domains depend only on data, computed once)
  dimensions.forEach(dim => {
    if (!("domain" in dim)) {
      dim.domain = d3_functor(dim.type.extent)(data.map(d => d[dim.key]))
    }
    if (!("scale" in dim)) {
      dim.scale = dim.type.defaultScale.copy()
    }
    dim.scale.domain(dim.domain)
  })

  // Build (or rebuild) the chart for the container's current width
  const build = () => {
    // Use clientWidth (layout pixels), not getBoundingClientRect(): Slidev
    // CSS-scales the slide, which would otherwise make the chart overflow.
    const width = containerEl.clientWidth - margin.left - margin.right

    // Skip while hidden/animating (e.g. when navigating between slides) or too narrow
    if (width < 50) return

    // Clear any previous render
    d3.select(containerEl).selectAll("svg, canvas").remove()

    const xscale = d3.scalePoint()
      .domain(d3.range(dimensions.length))
      .range([0, width])

    const svg = d3.select(containerEl).append("svg")
      .attr("width", width + margin.left + margin.right)
      .attr("height", height + margin.top + margin.bottom)
      .append("g")
      .attr("transform", `translate(${margin.left},${margin.top})`)

    const canvas = d3.select(containerEl).append("canvas")
      .attr("width", width * devicePixelRatio)
      .attr("height", height * devicePixelRatio)
      .style("width", width + "px")
      .style("height", height + "px")
      .style("margin-top", margin.top + "px")
      .style("margin-left", margin.left + "px")

    const ctx = canvas.node().getContext("2d")
    ctx.globalCompositeOperation = 'darken'
    ctx.globalAlpha = 0.15
    ctx.lineWidth = 1.5
    ctx.scale(devicePixelRatio, devicePixelRatio)

    const axes = svg.selectAll(".axis")
      .data(dimensions)
      .enter().append("g")
      .attr("class", d => "axis " + d.key.replace(/ /g, "_"))
      .attr("transform", (d, i) => `translate(${xscale(i)})`)

    const render = renderQueue(draw).rate(30)

    ctx.clearRect(0, 0, width, height)
    ctx.globalAlpha = d3.min([1.15 / Math.pow(data.length, 0.3), 1])
    render(data)

    axes.append("g")
      .each(function (d) {
        const renderAxis = "axis" in d
          ? d.axis.scale(d.scale)  // custom axis
          : yAxis.scale(d.scale)   // default axis
        d3.select(this).call(renderAxis)
      })
      .append("text")
      .attr("class", "title")
      .attr("text-anchor", "start")
      // sit the title in the top margin, steeply rotated so the closely
      // spaced labels stay readable and don't overlap as much
      .attr("transform", "translate(0,-9) rotate(-35)")
      .text(d => "description" in d ? d.description : d.key)

    // Add and store a brush for each axis.
    axes.append("g")
      .attr("class", "brush")
      .each(function (d) {
        d3.select(this).call(d.brush = d3.brushY()
          .extent([[-10, 0], [10, height]])
          .on("start", brushstart)
          .on("brush", brush)
          .on("end", brush)
        )
      })
      .selectAll("rect")
      .attr("x", -8)
      .attr("width", 16)

    svg.selectAll(".axis.pl_discmethod .tick text")
      .style("fill", color)

    function project(d) {
      return dimensions.map((p, i) => {
        // check if data element has property and contains a value
        if (!(p.key in d) || d[p.key] === null) return null
        return [xscale(i), p.scale(d[p.key])]
      })
    }

    function draw(d) {
      ctx.strokeStyle = color(d.pl_discmethod)
      ctx.beginPath()
      const coords = project(d)
      coords.forEach((p, i) => {
        // this tricky bit avoids rendering null values as 0
        if (p === null) {
          // render short horizontal stubs so sandwiched null values are visible
          if (i > 0) {
            const prev = coords[i - 1]
            if (prev !== null) {
              ctx.moveTo(prev[0], prev[1])
              ctx.lineTo(prev[0] + 6, prev[1])
            }
          }
          if (i < coords.length - 1) {
            const next = coords[i + 1]
            if (next !== null) ctx.moveTo(next[0] - 6, next[1])
          }
          return
        }
        if (i == 0) { ctx.moveTo(p[0], p[1]); return }
        ctx.lineTo(p[0], p[1])
      })
      ctx.stroke()
    }

    // d3 v7 passes the event to the handler (d3.event was removed)
    function brushstart(event) {
      if (event.sourceEvent) event.sourceEvent.stopPropagation()
    }

    // Handles a brush event, toggling the display of foreground lines.
    function brush() {
      render.invalidate()

      const actives = []
      svg.selectAll(".axis .brush")
        .filter(function (d) { return d3.brushSelection(this) })
        .each(function (d) {
          actives.push({ dimension: d, extent: d3.brushSelection(this) })
        })

      const selected = data.filter(d =>
        actives.every(active => {
          const dim = active.dimension
          // test if point is within extents for each active brush
          return dim.type.within(d[dim.key], active.extent, dim)
        })
      )

      ctx.clearRect(0, 0, width, height)
      ctx.globalAlpha = d3.min([0.85 / Math.pow(selected.length, 0.3), 1])
      render(selected)
    }
  }

  // ResizeObserver fires once the container gets a real size (after the slide
  // transition finishes) and again on any resize, so the chart always renders.
  resizeObserver = new ResizeObserver(() => build())
  resizeObserver.observe(containerEl)
})

onUnmounted(() => {
  if (resizeObserver) resizeObserver.disconnect()
})
</script>

<template>
  <div ref="containerRef" class="exoplanets-parcoords"></div>
</template>

<style>
.exoplanets-parcoords {
  position: relative;
  width: 110%;
  height: 360px;
  font: 10px sans-serif;
  /* shift the whole chart 50px to the left */
  transform: translateX(-50px);
}
/* Keep the discovery-method labels compact so the left margin stays tight. */
.exoplanets-parcoords .axis.pl_discmethod .tick text {
  font-size: 9px;
}
.exoplanets-parcoords > svg,
.exoplanets-parcoords > canvas {
  position: absolute;
  top: 0;
  left: 0;
}
/* The canvas overlays the SVG; let mouse events reach the brushes underneath. */
.exoplanets-parcoords > canvas {
  pointer-events: none;
}
.exoplanets-parcoords .axis .title {
  font-size: 11px;
  fill: #222;
  font-weight: bold;
}
.exoplanets-parcoords .axis line,
.exoplanets-parcoords .axis path {
  fill: none;
  stroke: #222;
  shape-rendering: crispEdges;
}
.exoplanets-parcoords .axis text {
  fill: #333;
  cursor: move;
}
.exoplanets-parcoords .brush .selection {
  fill-opacity: 0.3;
  stroke: #fff;
  shape-rendering: crispEdges;
}
</style>
