## Epoch - The Fastly Charting Library
By Ryan Sandor Richards

### Introduction

Epoch is a general purpose charting library for application developers and visualization designers. It focuses on two different aspects of
visualization programming: **basic charts** for creating historical reports, and **real-time charts** for displaying frequently 
updating timeseries data.

### Getting Started

1. [d3](https://github.com/mbostock/d3) and [jQuery](https://github.com/jquery/jquery) are required, so make sure you are including
them in your page.
2. Locate and download the `epoch.X.Y.Z.min.js` and `epoch.X.Y.Z.min.css` files in this repository, and place
them in your project (note: `X.Y.Z` is a placeholder for the current version of Epoch).
3. Include all the required JavaScript and CSS files into your source in the usual manner (probably in the head of the HTML document).
4. Read the examples and documentation below
5. Code, let cool, and enjoy (serves millions)

### A "quick & dirty" Introduction

Here are the basic steps you need to follow to create a multi-series area chart using Epoch:

**1) Include the appropriate scripts and styles**

```html
<head>
  <!-- include jquery and d3 HERE -->
  <script src="js/jquery.min.js"></script>
  <script src="js/d3.min.js"></script>
  <script src="js/epoch.X.Y.Z.min.js"></script>
  <link rel="stylesheet" type="text/css" href="css/epoch.X.Y.Z.min.css">
</head>
```

**2) Create the a container div for the chart**

```html
<div id="area" class="category10" style="width: 700px; height: 200px;"></div>
```

* Epoch will size the chart to fix the explicit dimensions of the container
* The class name "category10" refers to the categorical color scheme to use
  when rendering the chart. Other options are `category20` (default), 
  `category20b`, and `category20c`. See the 
  [d3 categorical color docs](https://github.com/mbostock/d3/wiki/Ordinal-Scales#categorical-colors)
  for more information.

**3) Setup the chart's data**

```javascript
var data = [
  { label: 'Layer 1', values: [ {x: 0, y: 0}, {x: 1, y: 1}, {x: 2, y: 2} ] },
  { label: 'Layer 2', values: [ {x: 0, y: 0}, {x: 1, y: 1}, {x: 2, y: 4} ] }
];
```

* Each chart type expects a certain data format. For the most part they are very similar to
  the example given above. Some types of charts (e.g. pie, guage, heatmap) require rather
  different formats. Make sure to read the chart-by-chart documentation below to see exactly
  what each type expects.

**4) Initialize, place, and draw the chart**

```javascript
var areaChartInstance = $('#area').epoch({ type: 'area', data: data });
```

* We use custom jQuery method `.epoch` to create the chart. It will automatically place it as child
  of the our container div and size it to fill the div completely.
* The `.epoch` function returns a programming interface with with to interact with the chart
  (in this example it is assigned to the `areaChartInstance` variable). For basic charts such as
  this it is used to update the chart's data, for example: `areaChartInstance.update(myNewData);`.

### Visual Styles

Epoch charts use CSS to set fill colors, strokes, etc. By default charts are colored using
[d3 categorical color](https://github.com/mbostock/d3/wiki/Ordinal-Scales#categorical-colors). You can easily override
these default colors or create your own custom categories.

#### Using Categorical Colors

We support the following categorical color sets:

* `category20` (default)
* `category20b`
* `category20c`
* `category10`

You can change a chart's color set by simply adding a class to the chart, like so:

```html
<div id="container1" class="epoch category20"></div>
<div id="container1" class="epoch category20b"></div>
<div id="container1" class="epoch category20c"></div>
<div id="container1" class="epoch category10"></div>
```

We achieve this by adding a category class to each element in a chart that needs to be rendered using the categorical color.

#### Creating Your Own Categories

The preferred method for doing this would be to use a css preprocessor like Sass or Less, here's an example of a simple color
scheme as written in SCSS:

```scss
$colors: red, green, blue, pink, yellow;

.epoch.my-colors {
  @for $i from 1 through 5 {
    .category#{$i} {
      .line { stroke: nth($colors, $i); }
      .area, .dot { fill: nth($colors, $i); }
    }
    .arc.category#{$i} path {
      fill: nth($colors, $i);
    }
    .bar.category#{$i} { 
      fill: nth($colors, $i);
    }
  }
}
```

You could then apply the class to your containers and see your colors in action:

```html
<div id="myChart" class="epoch my-colors"></div>
```

In the future we will be creating SCSS and LESS plugins with mixins you can use to more easily define custom color categories.


#### Specific Overrides

For multi-series charts, the data format requires that you supply a `label` for each series. We create a "dasherized"
class name from this label and associate it with the rendered output of the chart. For instance, take the following
data for an area chart:

```javascript
var areaChartData = [
  {
    label: "Layer 1",
    values: [ {x: 0, y: 100}, {x: 20, y: 1000} ]
  }
];
```

The layer labeled `Layer 1` will be associated in the chart with the class name `layer-1`. To override it's color simply
use the following CSS:

```css
#myChartContainer .layer-1 .area {
  fill: pink;
}
```


### Basic Charts

Epoch's **Basic** charts are implemented using d3 over a thin class hieracrchy. The classes perform common tasks (such as setting up
scales, axes, etc.) while the individual charts implement their own specialized drawing routines. 

Every basic chart was built to use the same workflow, here's an overview:

1. Create an HTML container.
  - Epoch automagically sizes charts to fit their containers.
  - `<div id="myChart" style="width: 200px; height: 200px"></div>`
2. Fetch and format your data.
  - Each type of chart uses a specific (though often familiar) data format.
  - See the documentation below for how each chart is expecting the data.
3. Use the jQuery method `.epoch` to create, append, and draw the chart.
  - `var myChart = $('#myChart').epoch({ type: 'line', data: myData });`
4. When you data changes, simply use the `update` method on the chart instance.
  - `myChart.update(myNewData);`

The rest of this section explains the individual charts in detail.


#### area

![alt tag](http://cdn.epoch.coffee.global.prod.fastly.net/images/basic/area.png)

The basic area chart is used to plot multiple data series atop one another. The chart expects data as an array of layers
each with their own indepenent series of values. To begin, let's take a look at some example data:

```javascript
var areaChartData = [
  // The first layer
  {
    label: "Layer 1",
    values: [ {x: 0, y: 100}, {x: 20, y: 1000}, ... ]
  },

  // The second layer
  {
    label: "Layer 2",
    values: [ {x: 0, y: 78}, {x: 20, y: 98}, ... ]
  },

  // Add as many layers as you would like!
];
```

As you can see the data is arranged as an array of layers. Each layer is an object that has the following properties:

* `label` - The name of the layer
* `values` - An array of values (each value having an `x` and `y` coordinate)

For the best results each layer should contain the same number of values, with the same `x` coordinates. This will allow
d3 to make the best looking graphs possible. To create a single series plot simply include a single layer.

Given that you have data in the appropriate format, instantiating a new chart is fairly easy. Simply create a container
element in HTML and use the jQuery bindings to create, place, and draw the chart:

```html
<div id="areaChart" style="width: 800px; height: 200px"></div>
<script>
  $('#areaChart').epoch({
    type: 'area',
    data: areaChartData
  });
</script>
```

In the `<script>` portion of the example above you'll notice that we are passing options to the `.epoch` method. The following
options are available for area charts:

* `axes` - Which axes to display.
  - Example: `axes: ['top', right', 'bottom', 'left']`
* `ticks` - Number of ticks to display on each axis.
  - Example: `{ top: 10, right: 5, bottom: 20, left: 5 }`
* `tickFormats` - What formatting function to use when displaying tick labels.
  - Example: `tickFormats: { bottom: function(d) { return '$' + d.toFixed(2); } }`
* `margins` - Explicit margin overrides for the chart.
  - Example: `margins: { top: 50, right: 30, bottom: 100, left: 40 }`
* `width` - Override automatic width with an explicit pixel value
  - Example: `width: 320`
* `height` - Override automatic height with an explicit pixel value
  - Example: `height: 240`


#### bar
![alt tag](http://cdn.epoch.coffee.global.prod.fastly.net/images/basic/bar.png)

Epoch's implementation of a multi-series grouped bar chart. Bar charts are useful for showing data by group over a discrete
domain. First, let's look at how the data is formatted for a bar chart:

```javascript
var barChartData = [
  // First bar series
  {
    label: 'Series 1',
    values: [
      { x: 'A', y: 30 },
      { x: 'B', y: 10 },
      { x: 'C', y: 12 },
      ...
    ]
  },

  // Second series
  {
    label: 'Series 2',
    values: [
      { x: 'A', y: 20 },
      { x: 'B', y: 39 },
      { x: 'C', y: 8 },
      ...
    ]
  },

  ... // Add as many series as you'd like
];
```

The bar chart will create groups of bars that share a like `x` value for each independent value present in the values array. 
Currently only grouped bar charts are available but we're planning on adding *stacked* and *normalized stacked* charts soon!

Next, let's take a look at the markup and scripting required to display our bar data:

```html
<div id="barChart" style="width: 300px; height: 100px"></div>
<script>
  $('#barChart').epoch({
    type: 'bar',
    data: barChartData
  });
</script>
```

The following options are available for bar charts:

* `axes` - Which axes to display.
  - Example: `axes: ['top', right', 'bottom', 'left']`
* `ticks` - Number of ticks to display on each axis.
  - Example: `{ top: 10, right: 5, bottom: 20, left: 5 }`
* `tickFormats` - What formatting function to use when displaying tick labels.
  - Example: `tickFormats: { bottom: function(d) { return '$' + d.toFixed(2); } }`
* `margins` - Explicit margin overrides for the chart.
  - Example: `margins: { top: 50, right: 30, bottom: 100, left: 40 }`
* `width` - Override automatic width with an explicit pixel value
  - Example: `width: 320`
* `height` - Override automatic height with an explicit pixel value
  - Example: `height: 240`


#### line
![alt tag](http://cdn.epoch.coffee.global.prod.fastly.net/images/basic/line.png)

Line charts are helpful for visualizing single or multi-series data when without stacking or shading. To begin, let's take a look
at the data format used by epoch's line chart:

```javascript
var lineChartData = [
  // The first series
  {
    label: "Series 1",
    values: [ {x: 0, y: 100}, {x: 20, y: 1000}, ... ]
  },

  // The second series
  {
    label: "Series 2",
    values: [ {x: 20, y: 78}, {x: 30, y: 98}, ... ]
  },

  ...
];
```

Notice that the data is very similar to that of the **area** chart above, with the exception that they need not cover the same domain
nor do the entries in each series have to line up via the `x` coordinate.

Next let's take a look at how you would implement the chart with markup and scripting:

```html
<div id="lineChart" style="width: 700px; height: 250px"></div>
<script>
  $('#lineChart').epoch({
    type: 'line',
    data: lineChartData
  });
</script>
```

The line charts supports the following options:

* `axes` - Which axes to display.
  - Example: `axes: ['top', right', 'bottom', 'left']`
* `ticks` - Number of ticks to display on each axis.
  - Example: `{ top: 10, right: 5, bottom: 20, left: 5 }`
* `tickFormats` - What formatting function to use when displaying tick labels.
  - Example: `tickFormats: { bottom: function(d) { return '$' + d.toFixed(2); } }`
* `margins` - Explicit margin overrides for the chart.
  - Example: `margins: { top: 50, right: 30, bottom: 100, left: 40 }`
* `width` - Override automatic width with an explicit pixel value
  - Example: `width: 320`
* `height` - Override automatic height with an explicit pixel value
  - Example: `height: 240`


#### Pie
![alt tag](http://cdn.epoch.coffee.global.prod.fastly.net/images/basic/pie.png)

Pie charts are useful for displaying the relative sizes of various data points. To begin, let's take a look at the data format
used by Epoch's pie chart implementation:


```javascript
var pieData = [
  { label: 'Slice 1', value: 10 },
  { label: 'Slice 2', value: 20 },
  { label: 'Slice 3', value: 40 },
  { label: 'Slice 4', value: 30 }
]
```

The data itself is an array of objects that represent the slices in the pie chart. The `label` parameter will be used to set
the label for the slice in the chart and the `value` parameter will be used to determine its visual size in comparison to the
other slices.

Once you have your data formatted correctly you can easly add a chart to your page using the following markup and script:

```html
<div id="pie" style="width: 400px; height: 400px"></div>
<script>
  $('#pie').epoch({
    type: 'pie',
    data: pieData
  });
</script>
```

The pie chart accepts the following parameters during initialization:

* `margin` - Surrounds the chart with a defined pixel margin
  - Example: `margin: 30`
* `inner` - Inner radius for the pie chart (for making Donut charts)
  - Example: `inner: 100`
* `width` - Override automatic width with an explicit pixel value
  - Example: `width: 320`
* `height` - Override automatic height with an explicit pixel value
  - Example: `height: 240`


#### Scatter
![alt tag](http://cdn.epoch.coffee.global.prod.fastly.net/images/basic/scatter.png)

Scatter plots are useful for visualizing statistical or sampling data in hopes of revealing patterns. To begin let's take a look at the
data format used by scatter plots:

```javascript
var scatterData = [
  // The first group
  {
    label: "Group 1",
    values: [ {x: 5, y: 100}, {x: 93, y: 1424}, ... ]
  },

  // The second group
  {
    label: "Group 2",
    values: [ {x: -52, y: 78}, {x: 120, y: 17}, ... ]
  },

  ...
];
```

The data is composed of an array containing "groups" of points. Groups need not have the same number of values nor do they
have to match the same `x` coordinate domains.

Next, let's see the markup and scripting needed to add the plot to your page:

```html
<div id="scatter" style="width: 500px; height: 500px"></div>
<script>
  $('#scatter').epoch({
    type: 'scatter',
    data: scatterData
  });
</script>
```

Scatter plots accept the following optional parameters:

* `radius` - How large the "dots" should be in the plot (in pixels)
  - Example: `radius: 4.5`
* `margins` - Explicit margin overrides for the chart.
  - Example: `margins: { top: 50, right: 30, bottom: 100, left: 40 }`
* `axes` - Which axes to display.
  - Example: `axes: ['top', right', 'bottom', 'left']`
* `ticks` - Number of ticks to display on each axis.
  - Example: `{ top: 10, right: 5, bottom: 20, left: 5 }`
* `tickFormats` - What formatting function to use when displaying tick labels.
  - Example: `tickFormats: { bottom: function(d) { return '$' + d.toFixed(2); } }`
* `width` - Override automatic width with an explicit pixel value
  - Example: `width: 320`
* `height` - Override automatic height with an explicit pixel value
  - Example: `height: 240`


### Real-time Charts

Epoch's **real-time** charts have been fine tuned for displaying frequently updating *timeseries* data. To make them performant (aka
not crash the browser) we've implemented the charts using a hybrid approach of d3 SVG with custom HTML5 Canvas rendering.

Every real-time chart has a name prefixed with `time.` and was built to use the same workflow, here's an overview:

1. Create an HTML container.
  - Epoch automagically sizes charts to fit their containers.
  - `<div id="myChart" style="width: 200px; height: 200px"></div>`
2. Fetch and format your data.
  - Each type of chart uses a specific (though often familiar) data format.
  - See the documentation below for how each chart is expecting the data.
3. Use the jQuery method `.epoch` to create, append, and draw the chart.
  - `var myChart = $('#myChart').epoch({ type: 'time.line', data: myData });`
4. When you have a new data point to append to the chart, use the `.push` method:
  - `myChart.push(nextDataPoint);`


#### Device Pixel Ratio

Epoch supports high resolution displays by automatically detecting and setting the appropriate pixel ratio for the canvas based real-time charts. You can override this behavior by explicitly setting the pixel ratio for any chart described below. Here's an example of how to do this:

```javascript
$('#my-chart').epoch({
  type: 'time.line',
  pixelRatio: 1
})
```

Note that the `pixelRatio` option must be an integer >= 1.


#### time.area
![alt tag](http://cdn.epoch.coffee.global.prod.fastly.net/images/time/area.png)

The real-time area chart works in a very similar way to the basic area chart detailed above. It is used to show relative sizes of
multi-series data as it evolves over time.

```javascript
var areaChartData = [
  // The first layer
  {
    label: "Layer 1",
    values: [ {time: 1370044800, y: 100}, {time: 1370044801, y: 1000}, ... ]
  },

  // The second layer
  {
    label: "Layer 2",
    values: [ {time: 1370044800, y: 78}, {time: 1370044801, y: 98}, ... ]
  },

  ...
];
```

As you can see the data is arranged as an array of layers. Each layer is an object that has the following properties:

* `label` - The name of the layer
* `values` - An array of values each value having a unix timestamp (`time`) and a value at that time (`y`)

The real-time chart requires that values in each layer have the exact same number of elements and that each corresponding
entry have the same `time` value.

Given that you have data in the appropriate format, instantiating a new chart is fairly easy. Simply create a container
element in HTML and use the jQuery bindings to create, place, and draw the chart:

```html
<div id="areaChart" style="width: 800px; height: 200px"></div>
<script>
  $('#areaChart').epoch({
    type: 'time.area',
    data: areaChartData
  });
</script>
```

In the `<script>` portion of the example above you'll notice that we are passing options to the `.epoch` method. The following
options are available for area charts:

* `axes` - Which axes to display.
  - Example: `axes: ['top', right', 'bottom', 'left']`
* `ticks` - Number of ticks to display on each axis.
  - Example: `{ time: 10, right: 5, left: 5 }`
* `tickFormats` - What formatting function to use when displaying tick labels.
  - Example: `tickFormats: { time: function(d) { return new Date(time*1000).toString(); } }`
* `fps` - Number of frames per second that transitions animations should use.
  - High values for this number are basically inpeceptible and can cause a big performance hit.
  - The default of `24` tends to work very well, but you can increase it to get smoother animations.
  - Example: `fps: 60`
* `windowSize` - Number of entries to display in the graph.
  - Example: `windowSize: 60` (shows a minute of by second data)
* `historySize` - Number of historical entries to hold at any time.
  - Example: `historySize: 240`
* `queueSize` - Number of entries to keep in working memory while the chart is not animating transitions.
  - Some browsers will not run animation intervals if a tab is inactive. This parameter allows you to
    bound the number of entries that have been recieved via the `.push` in this case (so as to reduce
    memory bloating).
  - Example: `queueSize: 120`
* `margins` - Explicit margin overrides for the chart.
  - Example: `margins: { top: 50, right: 30, bottom: 100, left: 40 }`
* `width` - Override automatic width with an explicit pixel value
  - Example: `width: 320`
* `height` - Override automatic height with an explicit pixel value
  - Example: `height: 240`
* `pixelRatio` - Override detected pixel ratio with an explicit value

#### time.bar
![alt tag](http://cdn.epoch.coffee.global.prod.fastly.net/images/time/bar.png)

The real-time bar chart is used to show relative sizes of multi-series data in the form of *stacked bars* as it evolves over time.

```javascript
var barChartData = [
  // First series
  {
    label: "Series 1",
    values: [ {time: 1370044800, y: 100}, {time: 1370044801, y: 1000}, ... ]
  },

  // The second series
  {
    label: "Series 2",
    values: [ {time: 1370044800, y: 78}, {time: 1370044801, y: 98}, ... ]
  },

  ...
];
```

As you can see the data is arranged as an array of layers. Each layer is an object that has the following properties:

* `label` - The name of the layer
* `values` - An array of values each value having a unix timestamp (`time`) and a value at that time (`y`)

The real-time chart requires that values in each layer have the exact same number of elements and that each corresponding
entry have the same `time` value.

Given that you have data in the appropriate format, instantiating a new chart is fairly easy. Simply create a container
element in HTML and use the jQuery bindings to create, place, and draw the chart:

```html
<div id="barChart" style="width: 800px; height: 200px"></div>
<script>
  $('#barChart').epoch({
    type: 'time.bar',
    data: barChartData
  });
</script>
```

In the `<script>` portion of the example above you'll notice that we are passing options to the `.epoch` method. The following
options are available for real-time bar charts:

* `axes` - Which axes to display.
  - Example: `axes: ['top', right', 'bottom', 'left']`
* `ticks` - Number of ticks to display on each axis.
  - Example: `{ time: 10, right: 5, left: 5 }`
* `tickFormats` - What formatting function to use when displaying tick labels.
  - Example: `tickFormats: { time: function(d) { return new Date(time*1000).toString(); } }`
* `fps` - Number of frames per second that transitions animations should use.
  - High values for this number are basically inpeceptible and can cause a big performance hit.
  - The default of `24` tends to work very well, but you can increase it to get smoother animations.
  - Example: `fps: 60`
* `windowSize` - Number of entries to display in the graph.
  - Example: `windowSize: 60` (shows a minute of by second data)
* `historySize` - Number of historical entries to hold at any time.
  - Example: `historySize: 240`
* `queueSize` - Number of entries to keep in working memory while the chart is not animating transitions.
  - Some browsers will not run animation intervals if a tab is inactive. This parameter allows you to
    bound the number of entries that have been recieved via the `.push` in this case (so as to reduce
    memory bloating).
  - Example: `queueSize: 120`
* `margins` - Explicit margin overrides for the chart.
  - Example: `margins: { top: 50, right: 30, bottom: 100, left: 40 }`
* `width` - Override automatic width with an explicit pixel value
  - Example: `width: 320`
* `height` - Override automatic height with an explicit pixel value
  - Example: `height: 240`
* `pixelRatio` - Override detected pixel ratio with an explicit value

#### time.line
![alt tag](http://cdn.epoch.coffee.global.prod.fastly.net/images/time/line.png)

The real-time line chart is used to display multi-series data as it changes over time. To begin, the chart uses the following
data format:

```javascript
var lineChartData = [
  // First series
  {
    label: "Series 1",
    values: [ {time: 1370044800, y: 100}, {time: 1370044801, y: 1000}, ... ]
  },

  // The second series
  {
    label: "Series 2",
    values: [ {time: 1370044800, y: 78}, {time: 1370044801, y: 98}, ... ]
  },

  ...
];
```

As you can see the data is arranged as an array of layers. Each layer is an object that has the following properties:

* `label` - The name of the layer
* `values` - An array of values each value having a unix timestamp (`time`) and a value at that time (`y`)

The real-time chart requires that values in each layer have the exact same number of elements and that each corresponding
entry have the same `time` value.

Given that you have data in the appropriate format, instantiating a new chart is fairly easy. Simply create a container
element in HTML and use the jQuery bindings to create, place, and draw the chart:

```html
<div id="lineChart" style="width: 800px; height: 200px"></div>
<script>
  $('#lineChart').epoch({
    type: 'time.bar',
    data: lineChartData
  });
</script>
```

In the `<script>` portion of the example above you'll notice that we are passing options to the `.epoch` method. The following
options are available for real-time bar charts:

* `axes` - Which axes to display.
  - Example: `axes: ['top', right', 'bottom', 'left']`
* `ticks` - Number of ticks to display on each axis.
  - Example: `{ time: 10, right: 5, left: 5 }`
* `tickFormats` - What formatting function to use when displaying tick labels.
  - Example: `tickFormats: { time: function(d) { return new Date(time*1000).toString(); } }`
* `fps` - Number of frames per second that transitions animations should use.
  - High values for this number are basically inpeceptible and can cause a big performance hit.
  - The default of `24` tends to work very well, but you can increase it to get smoother animations.
  - Example: `fps: 60`
* `windowSize` - Number of entries to display in the graph.
  - Example: `windowSize: 60` (shows a minute of by second data)
* `historySize` - Number of historical entries to hold at any time.
  - Example: `historySize: 240`
* `queueSize` - Number of entries to keep in working memory while the chart is not animating transitions.
  - Some browsers will not run animation intervals if a tab is inactive. This parameter allows you to
    bound the number of entries that have been recieved via the `.push` in this case (so as to reduce
    memory bloating).
  - Example: `queueSize: 120`
* `margins` - Explicit margin overrides for the chart.
  - Example: `margins: { top: 50, right: 30, bottom: 100, left: 40 }`
* `width` - Override automatic width with an explicit pixel value
  - Example: `width: 320`
* `height` - Override automatic height with an explicit pixel value
  - Example: `height: 240`
* `pixelRatio` - Override detected pixel ratio with an explicit value


#### time.gauge
![alt tag](http://cdn.epoch.coffee.global.prod.fastly.net/images/time/gauge.png)

The gauge chart is used to monitor values over a particular range as they change over time. The chart displays a gauge that is similar to
an automobile speedometer.

Unlike most Epoch charts the time gauge does not accept a `data` parameter upon initialization. Instead it has no *memory* and only knows
about its current `value`.

The real-time gauge chart is also programmed to fit a **specific aspect ratio** (4:3). To aid programmers we have included a few css
classes you can use to automatically size the chart:

* `gauge-tiny` (120px by 90px)
* `gauge-small` (180px by 135px)
* `gauge-medium` (240px by 180px)
* `gauge-large` (320px by 240px)

You do not have to use these classes, but you must ensure that your chart container is constrained to the 4 to 3 aspect ratio in order
for the chart to render correctly. Let's take a look at how one might create a new gauge chart:

```html
<div id="gaugeChart" class="epoch gauge-small"></div>
<script>
  $('#gaugeChart').epoch({
    type: 'time.gauge',
    value: 0.5
  });
</script>
```

The gauge chart can be configured for your particular use case using the following optional parameters:

* `domain` - The input domain when setting the value for the gauge chart.
  - Essentially this is an array with the first element representing the minimum value to expect
    and the second element representing the maximum value to expect.
  - Example: `domain: [0, 1]` (the chart should expect data between 0 and 1)
* `ticks` - Number of tick marks to display on the chart.
  - Example: `ticks: 10`
* `tickSize` - Size in pixels for each tick.
  - Example: `tickSize: 5`
* `tickOffset` - Number of pixels to offset ticks by from the outter arc of the gauge.
  - Example: `tickOffset: 10`
* `format` - The number formatter to use for the gauge's internal display label
  - Example: `format: function(v) { return (v*100).toFixed(2) + '%'; }`
* `fps` - Number of frames per second that transitions animations should use.
  - High values for this number are basically inpeceptible and can cause a big performance hit.
  - The default of `24` tends to work very well, but you can increase it to get smoother animations.
  - Example: `fps: 60`
* `pixelRatio` - Override detected pixel ratio with an explicit value


#### time.heatmap
![alt tag](http://cdn.epoch.coffee.global.prod.fastly.net/images/time/heatmap.png)

The real-time heatmap chart is used to visualize normalized histogram data over time. It works by first sorting incoming histograms
into a small set of discrete buckets. For multi-series data it uses color blending to show series concentration.

This type of chart has the most "intense" data format, with each entry expecting a sparse histogram. Let's examine this format using
and example:

```javascript
var heatmapData = [
  // First Series
  {
    label: 'Series 1',
    values: [
      {
        time: 1370044800,
        histogram: {
          18: 49,
          104: 10,
          ...
        }
      },

      {
        time: 1370044801,
        histogram: {
          9: 8,
          120: 17,
          ...
        }
      },
    ]
  },

  ...
];
```

As you can see the data is arranged as an array of layers. Each layer is an object that has the following properties:

* `label` - The name of the layer
* `value` - A list of unix timestamp associated sparse histograms:
  - `time` - The unix timestam for the entry
  - `histogram` - A "sparse" frequency hash that maps values to frequencies

Given that you have data in the appropriate format, instantiating a new chart is fairly easy. Simply create a container
element in HTML and use the jQuery bindings to create, place, and draw the chart:

```html
<div id="heatmapChart" style="width: 800px; height: 200px"></div>
<script>
  $('#heatmapChart').epoch({
    type: 'time.heatmap',
    data: heatmapData
  });
</script>
```

The heatmap is one of the most configurable of all Epoch charts, here's a run down of the available options:

* `buckets` - Number of buckets to display in each column of the visible chart window.
  - Example: `buckets: 20`
* `bucketRange` - The range covered by the buckets.
  - Note: values that fall above the range will be placed in the top most bucket, and
    values that fall below the range will be placed in the bottom must bucker.
  - Example: `bucketRange: [0, 1000]`
* `bucketPadding` - Amount of padding to place around buckets in the display
  - Example: `bucketPadding: 0`
* `opacity` - The opacity function to use when rendering buckets
  - Each bucket will be rendered using a specific opacity. More saturated colors represent higher values in
    histogram and more transparent colors represent lower values.
  - There are many built-in opacity functions: `root`, `linear`, `quadratic`, `cubic`, `quartic`, and `quintic`.
  - You can define your own custom function, see the example below.
  - Example: `opacity: function(value, max) { return Math.pow(value/max, 0.384); }`
* `axes` - Which axes to display.
  - Example: `axes: ['top', right', 'bottom', 'left']`
* `ticks` - Number of ticks to display on each axis.
  - Example: `{ time: 10, right: 5, left: 5 }`
* `tickFormats` - What formatting function to use when displaying tick labels.
  - Example: `tickFormats: { time: function(d) { return new Date(time*1000).toString(); } }`
* `fps` - Number of frames per second that transitions animations should use.
  - High values for this number are basically inpeceptible and can cause a big performance hit.
  - The default of `24` tends to work very well, but you can increase it to get smoother animations.
  - Example: `fps: 60`
* `windowSize` - Number of entries to display in the graph.
  - Example: `windowSize: 60` (shows a minute of by second data)
* `historySize` - Number of historical entries to hold at any time.
  - Example: `historySize: 240`
* `queueSize` - Number of entries to keep in working memory while the chart is not animating transitions.
  - Some browsers will not run animation intervals if a tab is inactive. This parameter allows you to
    bound the number of entries that have been recieved via the `.push` in this case (so as to reduce
    memory bloating).
  - Example: `queueSize: 120`
* `margins` - Explicit margin overrides for the chart.
  - Example: `margins: { top: 50, right: 30, bottom: 100, left: 40 }`
* `width` - Override automatic width with an explicit pixel value
  - Example: `width: 320`
* `height` - Override automatic height with an explicit pixel value
  - Example: `height: 240`
* `pixelRatio` - Override detected pixel ratio with an explicit value

### Developing Epoch

To work on the epoch charting library itself do the following:

1. Clone the repository
2. In the clone, `npm install`
3. then `cake build`.
4. View `docs/static.html` and `docs/time.html` for a quick overview of the available features
5. Scour the source for more detailed information (formal documentation coming soon)
6. Let cool and enjoy (serves millions).
7. Keep the documentation up-to-date run `codo` from the command line
