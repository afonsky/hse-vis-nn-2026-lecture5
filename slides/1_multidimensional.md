---
layout: center
---

# Визуализация многомерных данных

---

# Данные малой размерности (1D)

<br>
<br>

<center>
<div>
<v-plotly style="width: 400px !important; height: 200px !important"
:data="[{
x: Array.from({length: 15}, () => Math.random()*4.5),
y: Array.from({length: 15}, () => Math.random()*0),
type: 'scatter',
mode: 'markers',
marker: {color: 'red', size: 10, opacity: 0.5},
showlegend: false
},
{
x: Array.from({length: 15}, () => Math.random()*4.5+5.5),
y: Array.from({length: 15}, () => Math.random()*0),
type: 'scatter',
mode: 'markers',
marker: {color: 'green', size: 10, opacity: 0.5},
showlegend: false
},
{
x: [5.0],
y: [0],
type: 'scatter',
mode: 'markers',
marker: {color: 'blue', size: 10, symbol: 'cross'},
showlegend: false
}]"
:layout="{
xaxis: {zeroline: false},
yaxis: {showticklabels: false, showgrid: false},
margin: {l: 10, r:50, pad: 1}
}"
:config="{displayModeBar: false}"
:options="{}"/>
</div>
</center>

---

# Данные малой размерности (2D)

<br>

<center>
<div>
<v-plotly style="width: 400px !important; height: 400px !important"
:data="[{
x: Array.from({length: 25}, () => Math.random()*0.45),
y: Array.from({length: 25}, () => Math.random()*0.45),
type: 'scatter',
mode: 'markers',
marker: {color: 'red', size: 10, opacity: 0.5},
showlegend: false
},
{
x: Array.from({length: 15}, () => Math.random()*0.55+0.5),
y: Array.from({length: 15}, () => Math.random()*0.55+0.5),
type: 'scatter',
mode: 'markers',
marker: {color: 'green', size: 10, opacity: 0.5},
showlegend: false
},
{
x: [0, 0.9],
y: [1.05, 0],
type: 'scatter',
mode: 'lines',
line: {color: 'blue'},
showlegend: false
}]"
:layout="{
xaxis: {title: 'x<sub>1</sub>'},
yaxis: {title: 'x<sub>2</sub>'},
margin: {l: 40, r:20, b:70, t:20, pad: 2}
}"
:config="{displayModeBar: false}"
:options="{}"/>
</div>
</center>

---

# Данные малой размерности (3D)

<br>

<center>
<div>
<v-plotly style="height: 400px; position: relative"
:data="[{
x: Array.from({length: 25}, () => Math.random()*0.5),
y: Array.from({length: 25}, () => Math.random()*0.5),
z: Array.from({length: 25}, () => Math.random()*0.5),
type: 'scatter3d',
mode: 'markers',
marker: {color: 'red', size: 4, opacity: 0.5},
showlegend: false
},
{
x: Array.from({length: 15}, () => Math.random()*0.6+0.5),
y: Array.from({length: 15}, () => Math.random()*0.6+0.5),
z: Array.from({length: 15}, () => Math.random()*0.6+0.5),
type: 'scatter3d',
mode: 'markers',
marker: {color: 'green', size: 4, opacity: 0.5},
showlegend: false
},
{
x: [0,1,1,0],
y: [0.9,0.4,0.2,0.7],
z: [0,0,1.0,1.0],
i: [0,0,0,1],
j: [1,2,3,2],
k: [2,3,1,3],
type: 'mesh3d',
color: 'blue',
opacity: 0.35,
showlegend: false
}]"
:layout="{
   scene: {camera: {eye: {x: 1.75, y: -1.25, z:1.05}},
            xaxis: {title: 'x<sub>1</sub>', range: [0.01,1]},
            yaxis: {title: 'x<sub>2</sub>', range: [0.,1]},
            zaxis: {title: 'x<sub>3</sub>', range: [0.,1]}},
   margin: {l: 20, r:20, b:20, t:1, pad: 5},
}"
:config="{displayModeBar: false}"
:options="{}"/>
</div>
</center>

---

# Множественное сопоставление (см. [Лекцию 4](https://afonsky.github.io/hse-vis-nn-2026-lecture4))


<div class="grid grid-cols-[5fr_5fr] gap-16">
<div>
<figure>
  <img src="/scatterplot_D3.png" style="width: 400px !important;">
</figure>
</div>
<div>
<br>
<figure>
  <img src="/partitioning_1.svg" style="width: 360px !important;">
    <figcaption style="color:#b3b3b3ff; font-size: 11px; position: relative; top: 20px; left: 80px;">Источники изображений:<br> <a href="https://observablehq.com/@d3/splom/2">https://observablehq.com/@d3/splom/2</a> (слева)<br> <a href="https://gist.github.com/mbostock/3887051">https://gist.github.com/mbostock/3887051</a> (справа)
    </figcaption>
</figure>
</div>
</div>

---
zoom: 0.92
---

# Множественное сопоставление со взаимодействием

<div class="mt-12">
<splom />
</div>

---

# Параллельные координаты (здесь 4D)

<div class="grid grid-cols-[2fr_10fr] gap-1">
<div>
<figure>
    <img src="/iris_setosa.png" style="width: 135px !important;">
    <img src="/iris_versicolor.png" style="width: 135px !important;">
    <img src="/iris_virginica.png" style="width: 135px !important;">
</figure>
</div>
<div class="mt-12">
<parallel_basic />
</div>
</div>

<span style="color:#b3b3b3ff; font-size: 11px; position: relative; top: 0px; left: 0px;">Источник фотографий: <span class="external-link" role="link" tabindex="0" onclick="window.open('https://en.wikipedia.org/wiki/Iris_flower_data_set','_blank')">https://en.wikipedia.org/wiki/Iris_flower_data_set</span></span> <span style="color:#b3b3b3ff; font-size: 11px; position: relative; top: 0px; left: 80px;">Источник визуализации: <span class="external-link" role="link" tabindex="0" onclick="window.open('https://d3-graph-gallery.com/graph/parallel_basic.html','_blank')">https://d3-graph-gallery.com/graph/parallel_basic.html</span></span>

---

# Параллельные координаты

#### Метод параллельных координат представляет собой эффективный способ визуализации нескольких переменных, а также связанных с ними **кластеров**, <span style="color:#3BADEF">**выбросов**</span>, <span style="color:#46B54A">**распределений значений**</span> и <span style="color:#f6931e">**корреляций**</span>.
<br>
<center>
<figure>
    <img src="/parallel_coordinate_explained.png" style="width: 735px !important;">
<figcaption style="color:#b3b3b3ff; font-size: 11px; position: relative; top: -25px; left: 80px;">Источник: <a href="https://github.com/eamonnmag/Visualization-Course">Eamonn Maguire</a>
    </figcaption>
</figure>
</center>

---

# Параллельные координаты

<div class="mt-12">
<parallel_custom />
</div>

<br>

<span style="color:#b3b3b3ff; font-size: 11px; position: relative; top: 0px; left: 0px;">Источник фотографий: <span class="external-link" role="link" tabindex="0" onclick="window.open('https://en.wikipedia.org/wiki/Iris_flower_data_set','_blank')">https://en.wikipedia.org/wiki/Iris_flower_data_set</span></span> <span style="color:#b3b3b3ff; font-size: 11px; position: relative; top: 0px; left: 80px;">Визуализация основана на <span class="external-link" role="link" tabindex="0" onclick="window.open('https://d3-graph-gallery.com/graph/parallel_custom.html','_blank')">https://d3-graph-gallery.com/graph/parallel_custom.html</span></span>

---

# Параллельные координаты (здесь 17D)

<div class="mt-12">
<exoplanets />
</div>

<span style="color:#b3b3b3ff; font-size: 11px; position: relative; top: 0px; left: 80px;">Визуализация основана на <span class="external-link" role="link" tabindex="0" onclick="window.open('https://gist.github.com/syntagmatic/482706e0638c67836d94b20f0cb37122','_blank')">https://gist.github.com/syntagmatic/482706e0638c67836d94b20f0cb37122</span></span>

---

# Диаграмма [Санкея](https://en.wikipedia.org/wiki/Matthew_Henry_Phineas_Riall_Sankey)

<div class="mt-2">
<sankey />
</div>

<span style="color:#b3b3b3ff; font-size: 11px; position: relative; top: -10px; left: 0px;">Источник визуализации: <span class="external-link" role="link" tabindex="0" onclick="window.open('https://observablehq.com/@d3/parallel-sets','_blank')">https://observablehq.com/@d3/parallel-sets</span></span>

---

# Потоковый график

<div class="mt-6">
<streamgraph />
</div>

<span style="color:#b3b3b3ff; font-size: 11px; position: relative; top: -10px; left: 0px;">Источник визуализации: <span class="external-link" role="link" tabindex="0" onclick="window.open('https://d3-graph-gallery.com/graph/streamgraph_template.html','_blank')">https://d3-graph-gallery.com/graph/streamgraph_template.html</span></span>