---
jupyter:
  jupytext:
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.2'
      jupytext_version: 1.3.3+dev
  kernelspec:
    display_name: .NET (C#)
    language: C#
    name: .net-csharp
---

# Charts with XPlot
Charts can be rendered using [Xplot.Plotly](https://fslab.org/XPlot/). 
We will cover some example on how to use XPlot in a notebook with the .NET Kernel.

First, import the `XPlot.Plotly` namespace:

```csharp
using XPlot.Plotly;
```

The next cell sets up some helpers for data generation.

```csharp
var generator = new Random();
```

# Rendering Scatter plots
One of the most commonly used type of chart to explore data set. Use the type `Scatter`.

```csharp
var openSeries = new Graph.Scatter
{
    name = "Open",
    x = new[] {1, 2, 3, 4},
    y = new[] {10, 15, 13, 17}
};

var closeSeries = new Graph.Scatter
{
    name = "Close",
    x = new[] { 2,3,4,5 },
    y = new[] { 16, 5, 11, 9 }
};

var chart = Chart.Plot(new[] {openSeries, closeSeries});
chart.WithTitle("Open vs Close");
display(chart);
```

Let's change it to be markers style, so more like a scatter plot.

```csharp
openSeries.mode = "markers";
closeSeries.mode = "markers";
chart = Chart.Plot(new [] {openSeries, closeSeries});
display(chart);
```

`Scatter` can also produce polar charts by setting the radial property `r` and angular proeprty `t`

```csharp
openSeries = new Graph.Scatter
{
    name = "Open",
    r = new[] {1, 2, 3, 4},
    t = new[] {45, 100, 150, 290}
};

closeSeries = new Graph.Scatter
{
    name = "Close",
    r = new[] { 2,3,4,5 },
    t = new[] { 16, 45, 118, 90 }
};

chart = Chart.Plot(new[] {openSeries, closeSeries});
chart.WithLayout(new Layout.Layout
            {
                orientation = -90
            });
display(chart);
```

## Large scatter plots and performance
It is not uncommong to have scatter plots with a large dataset, it is a common scenario at the beginning of a data exploration process. Using the default `svg` based rendering will create performace issues as the dom will become very large.
We can then use `web-gl` support to address the problem.

```csharp
#!time
var series = new Graph.Scattergl[10];

for(var i = 0; i < series.Length; i++){
    series[i] =  new Graph.Scattergl
    {
        name = $"Series {i}",
        mode = "markers",
        x = Enumerable.Range(0,100000).Select(_ => generator.Next(-200, 200) * 1000 * generator.Next(-2000, 2000)),
        y = Enumerable.Range(0,100000).Select(_ => generator.Next(-200, 200) * 1000 * generator.Next(-2000, 2000))
    };
}

chart = Chart.Plot(series);
chart.WithTitle("Large Dataset");
display(chart);
```

Can provide custom marker `colour`, `size` and `colorscale` to display even more information to the user.

```csharp
for(var i = 0; i < series.Length; i++){
    var sizes = Enumerable.Range(0,100).Select(_ => (generator.NextDouble() < 0.75) ? generator.Next(1, 5) : generator.Next(10, 15)).ToArray();
    var temperatures = sizes.Select(t => (t * 10) - 100);
    
    series[i].x = Enumerable.Range(0,100).Select(_ => generator.Next(-200, 200) * 1000 * generator.Next(-2000, 2000));
    series[i].y = Enumerable.Range(0,100).Select(_ => generator.Next(-200, 200) * 1000 * generator.Next(-2000, 2000));
    series[i].marker = new Graph.Marker{        
        size = sizes,
        color = temperatures,
        colorscale = "hot"    
    };
}

chart = Chart.Plot(series);
chart.WithTitle("Size and Colour");
display(chart);
```

Plotly pvoides some additional `color scales` to use.

```csharp
for(var i = 0; i < series.Length; i++) {
    series[i].marker.colorscale = "Viridis";
}

chart = Chart.Plot(series);
chart.WithTitle("Viridis scale");
display(chart);

for(var i = 0; i < series.Length; i++) {
    series[i].marker.colorscale = "Hot";
}

chart = Chart.Plot(series);
chart.WithTitle("Hot scale");
display(chart);

for(var i = 0; i < series.Length; i++) {
    series[i].marker.colorscale = "Jet";
}

chart = Chart.Plot(series);
chart.WithTitle("Jet scale");
display(chart);
```

# Rendering Histograms
Let's have a look at using histograms, the next cell sets up some generators.

```csharp
var count = 20;
var dates = Enumerable.Range(0, count).Select(i => DateTime.Now.AddMinutes(generator.Next(i, i + 30))).ToArray();
```

Now let's define histogram traces:

```csharp
var openByTime = new Graph.Histogram
{
    x = dates,
    y = Enumerable.Range(0, count).Select(_ => generator.Next(0,200)),
    name = "Open"
};

var closeByTime = new Graph.Histogram
{
    x = dates,
    y = Enumerable.Range(0, count).Select(_ => generator.Next(0, 200)),
    name = "Close"
};
chart = Chart.Plot(new [] {openByTime, closeByTime});
display(chart);
```

The Histogram generator will automatically count the number of items per bin. 

Setting `histfunc` to `"sum"` we can now add up all the values contained in each bin.
Note that we are creatng bin using the `x` data point and we are using bydefault autobinx

```csharp
openByTime.histfunc = "sum";;
closeByTime.histfunc = "sum";
chart = Chart.Plot(new [] {openByTime, closeByTime});
display(chart);
```

# Area chart and Polar Area chart


By populating hte property `fill` of a `Scatter` trace the chart will render as area chart.

Here is set to `"tozeroy"` which will create a fill zone underneath the line reachin to the 0 of the y axis.

```csharp
openSeries = new Graph.Scatter
{
    name = "Open",
    x = new[] {1, 2, 3, 4},
    y = new[] {10, 15, 13, 17},
    fill = "tozeroy",
    mode= "lines"
};

closeSeries = new Graph.Scatter
{
    name = "Close",
    x = new[] {1, 2, 3, 4},
    y = new[] {3, 5, 11, 9},
    fill = "tozeroy",
    mode= "lines"
};

chart = Chart.Plot(new[] {openSeries, closeSeries});
chart.WithTitle("Open vs Close");
display(chart);
```

With one `fill` set to `"tonexty"` the cahrt will fill the aread between traces.

```csharp
openSeries.fill = null;
closeSeries.fill = "tonexty";

chart = Chart.Plot(new[] {openSeries, closeSeries});
chart.WithTitle("Open vs Close");
display(chart);
```

Using `Area` traces we can generate radial area chart. In this example we are using cardinal points to xpress angular values.
The array `{"North", "N-E", "East", "S-E", "South", "S-W", "West", "N-W"}` will be autoimatically translated to angular values.

```csharp
var areaTrace1 =
    new Graph.Area
    {
        r = new[] {77.5, 72.5, 70.0, 45.0, 22.5, 42.5, 40.0, 62.5},
        t = new[] {"North", "N-E", "East", "S-E", "South", "S-W", "West", "N-W"},
        name = "11-14 m/s",
        marker = new Graph.Marker
        {
            color = "rgb(106,81,163)"
        }
    };

var areaTrace2 =
    new Graph.Area
    {
        r = new  [] {57.49999999999999, 50.0, 45.0, 35.0, 20.0, 22.5, 37.5, 55.00000000000001},
        t = new [] {"North", "N-E", "East", "S-E", "South", "S-W", "West", "N-W"},
        name = "8-11 m/s",
        marker = new Graph.Marker{
            color = "rgb(158,154,200)"
        }
    };

var areaTrace3 =
    new Graph.Area
    {
        r = new [] {40.0, 30.0, 30.0, 35.0, 7.5, 7.5, 32.5, 40.0},
        t = new [] {"North", "N-E", "East", "S-E", "South", "S-W", "West", "N-W"},
        name = "5-8 m/s",
        marker = new Graph.Marker{
            color = "rgb(203,201,226)"
        }
    };

var areaTrace4 =
    new Graph.Area
    {
        r = new [] {20.0, 7.5, 15.0, 22.5, 2.5, 2.5, 12.5, 22.5},
        t = new [] {"North", "N-E", "East", "S-E", "South", "S-W", "West", "N-W"},
        name = "< 5 m/s",
        marker = new  Graph.Marker{
            color = "rgb(242,240,247)"
        }
    };

var areaLayout = new Layout.Layout
{
    title = "Wind Speed Distribution in Laurel, NE",
    font = new Graph.Font
    {
        size = 16
    },
    legend = new Graph.Legend
    {
        font = new Graph.Font
        {
            size = 16
        }
    },
    radialaxis = new Graph.Radialaxis
    {
        ticksuffix = "%"

    },
    orientation = 270
};

chart = Chart.Plot(new [] {areaTrace1, areaTrace2, areaTrace3, areaTrace4});
chart.WithLayout(areaLayout);
display(chart);
```
