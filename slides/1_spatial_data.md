---
layout: center
---
# Визуализация пространственных данных

---

# Геоданные: фоновая картограмма (Choropleth)

<br>
<figure>
  <img src="/textbook/fig_8.2.svg" style="width: 600px !important;">
<figcaption style="color:#b3b3b3ff; font-size: 11px; position: relative; top: 20px; left: 0px;">Источник изображения:
<a href="https://www.cs.ubc.ca/~tmm/vadbook/">Рис. 8.2. T. Munzner. Visualization Analysis and Design (2014)</a></figcaption>
</figure>

---

# Скалярное поле

* Срез (слева)
* Полупрозрачные изоповерхности (посередине)
* Рендер поверхности (справа)
<figure>
  <img src="/scalar_field.png" style="width: 900px !important;">
<figcaption style="color:#b3b3b3ff; font-size: 11px; position: relative; top: -400px; left: 600px;">Источник изображений: Рис. 1.2 и 2.1b из статьи Kniss (2002)</figcaption>
</figure>

---

# Изолинии (контурные линии)

<figure>
  <img src="/isolines.png" style="width: 900px !important;">
<figcaption style="color:#b3b3b3ff; font-size: 11px; position: relative; top: -500px; left: 600px;">Источник изображения:<br> <a href="https://data.linz.govt.nz/layer/768-nz-mainland
-contours-topo-150k">https://data.linz.govt.nz/layer/<br>768-nz-mainland
-contours-topo-150k</a></figcaption>
</figure>

---
zoom: 0.75
--- 

# Изоповерхности

<figure>
  <img src="/Carr_2004.png" style="width: 1250px !important;">
<figcaption style="color:#b3b3b3ff; font-size: 11px; position: relative; top: 10px; left: 400px;">Источник изображений: <a href="https://www.cs.ubc.ca/~van/papers/2004-viz-contourtree.pdf">Рис. 1 из статьи "Simplifying Flexible Isosurfaces Using Local Geometric Measures" H. Carr et al. (2004)</a></figcaption>
</figure>
<br>

* Данные: 109 × 256 × 256 МРТ срезов (набор данных UNC Head)
* Слева изоповерхность показывает в основном череп
  * Дерево контуров слишком велико (1 573 373 ребра)
* Справа - изоповерхность, выбранная из упрощенного дерева контуров
  * Самая правая панель управляет степенью упрощения дерева контуров (92 ребра)

---
zoom: 0.8
--- 

# Многомерные передаточные функции

<br>
<div class="grid grid-cols-[5fr_5fr] gap-20">
<div>
<figure>
  <img src="/transfer_functions_1.png" style="width: 300px !important;">
</figure>
</div>
<div>
<figure>
  <img src="/transfer_functions_2.png" style="width: 250px !important;">
</figure>
</div>
</div>
<figure>
    <figcaption style="color:#b3b3b3ff; font-size: 11px; position: relative; top: -20px; left: 820px;">Изображения: <a href="https://viz.wtf/">Рис. 9.1 из статьи Kniss et al. (2005)</a>
  </figcaption>
</figure>

#### Можно использовать многомерные передаточные функции для прямого объемного рендеринга с использованием производных пространств
* Стандартная одномерная гистограмма может отображать три основных материала:<br> A - воздух, B - мягкие ткани и C - кости
* Полное двумерное производное пространство позволяет также различать границы материалов
* Объемный рендеринг набора данных головы с использованием полученной двумерной передаточной функции, показывающей границы материалов D - воздух-ткань, E - ткань-кость и F - воздух-кость

---

# Векторные поля

<br>
<div class="grid grid-cols-[7fr_5fr] gap-10">
<div>

* Задачи:
  * Поиск критических точек
  * Определение типа критической точки в конкретном месте
  * Предсказание того, куда попадет частица, стартующая из заданной точки (адвекция)
</div>
<div>
<figure>
  <img src="/textbook/fig_8.7.svg" style="width: 300px !important;">
  <figcaption style="color:#b3b3b3ff; font-size: 11px; position: relative; top: 20px; left: 0px;">Типы критических точек
</figcaption>
</figure>

<br>
<br>
<figure>
  <img src="/vector_fields.png" style="width: 350px !important;">
</figure>
</div>
</div>
<figure>
<figcaption style="color:#b3b3b3ff; font-size: 11px; position: relative; top: -20px; left: 0px;">Источник изображений:
<a href="https://www.cs.ubc.ca/~tmm/vadbook/">Рис. 8.7 и 8.8. T. Munzner. Visualization Analysis and Design (2014)</a></figcaption>
</figure>

---

# Линии тока и векторные поля в 3D

#### Преимущества 3D перевешивают недостатки, когда задача - восприятие формы трехмерных пространственных данных

<br>

<figure>
  <img src="/3D_streamline.png" style="width: 700px !important;">
<figcaption style="color:#b3b3b3ff; font-size: 11px; position: relative; top: -40px; left: 500px;">Источник изображения:<br>
<a href="https://people.freedesktop.org/~marcheu/vis10-streamlines.pdf">Stephane Marchesin, Cheng-Kai Chen, Chris Ho, and Kwan-Liu Ma (2010)</a></figcaption>
</figure>

---

# Геометрический поток

<div class="grid grid-cols-[7fr_3fr] gap-1">
<div>
<figure>
  <img src="/geometric_flow_1.png" style="width: 700px !important;">
</figure>
</div>
<div>
<figure>
  <img src="/geometric_flow_2.png" style="width: 187px !important;">
<figcaption style="color:#b3b3b3ff; font-size: 11px; position: relative; top: 0px; left: -10px;">Источник изображений:
<a href="https://www.cs.ubc.ca/~tmm/vadbook/">Рис. 8.9. T. Munzner. Visualization Analysis and Design (2014)</a></figcaption>
</figure>
</div>
</div>

<br>
<br>


#### Геометрические потоки показывают разреженный набор траекторий частиц
* Потоковые линии: все кластеры одинаково непрозрачны; синий кластер непрозрачен; красный кластер непрозрачен
* Траектории, окрашенные по трем кластерам (крайне правое изображение)