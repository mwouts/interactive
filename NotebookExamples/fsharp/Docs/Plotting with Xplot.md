---
jupyter:
  jupytext:
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.2'
      jupytext_version: 1.3.3+dev
  kernelspec:
    display_name: .NET (F#)
    language: F#
    name: .net-fsharp
---

# Charts with XPlot
Charts can be rendered using [Xplot.Plotly](https://fslab.org/XPlot/). 
We will cover some example on how to use XPlot in a notebook with the .NET Kernel.

First, import the `XPlot.Plotly` namespace:

```fsharp
open XPlot.Plotly
```

The next cell sets up some helpers for data generation.

```fsharp
let generator = new Random()
```

# Rendering Scatter plots
One of the most commonly used type of chart to explore data set. Use the type `Scatter`.

```fsharp
let openSeries =
    Scatter(
        name = "Open",
        x = [1; 2; 3; 4],
        y = [10; 15; 13; 17])

let closeSeries =
    Scatter(
        name = "Close",
        x = [2; 3; 4; 5],
        y = [16; 5; 11; 9])

[openSeries; closeSeries]
|> Chart.Plot
|> Chart.WithTitle "Open vs Close"
```

Let's change it to be markers style, so more like a scatter plot.

```fsharp
openSeries.mode <- "markers"
closeSeries.mode <- "markers"

[openSeries; closeSeries]
|> Chart.Plot
```

`Scatter` can also produce polar charts by setting the radial property `r` and angular proeprty `t`

```fsharp
let openSeries =
    Scatter(
        name = "Open",
        r = [1.; 2.; 3.; 4.],
        t = [45.; 100.; 150.; 290.])

let closeSeries =
    Scatter(
        name = "Close",
        r = [2.; 3.; 4.; 5. ],
        t = [16.; 45.; 118.; 90.])

[openSeries; closeSeries]
|> Chart.Plot
|> Chart.WithLayout(Layout(orientation = -90.))
```

## Large scatter plots and performance
It is not uncommong to have scatter plots with a large dataset, it is a common scenario at the beginning of a data exploration process. Using the default `svg` based rendering will create performace issues as the dom will become very large.
We can then use `web-gl` support to address the problem.

```fsharp
#!time

let series =
    [|
        for a in 1 .. 10 ->
            Scattergl(
                name = sprintf "Series %i" a, 
                mode = "markers", 
                x = [ for ax in 1 .. 100000 -> generator.Next(-200, 200) * 1000 * generator.Next(-2000, 2000)],     
                y = [ for ay in 1 .. 100000 -> generator.Next(-200, 200) * 1000 * generator.Next(-2000, 2000)])
    |]
    
series
|> Chart.Plot
|> Chart.WithTitle "Large Dataset"
```

Can provide custom marker `colour`, `size` and `colorscale` to display even more information to the user.

```fsharp
let generatePoint () = generator.Next(-200, 200) * 1000 * generator.Next(-2000, 2000)
    
let sizes =
    [ for s in 1..100 ->
        if generator.NextDouble() < 0.75 then
            generator.Next(1, 5)
         else
             generator.Next(10, 15) ]
    
let temperatures = sizes |> Seq.map (fun x -> x * 10 - 100)

let series =
    [|
        for a in 1 .. 10 -> 
            Scattergl(
                name = sprintf "Series %i" a,
                mode = "markers",
                x = [ for _ in 1 .. 100 -> generatePoint () ],
                y = [ for ay in 1 .. 100 -> generatePoint () ],
                marker = 
                    Marker(
                        colorscale = "hot",
                        color = temperatures,
                        size = sizes))
    |]
    
series
|> Chart.Plot
|> Chart.WithTitle "Size and Colour"
```

Plotly provides some additional `color scales` to use. Note that we use `display` explicitly to display each graph with separate titles, rather than a single chart.

```fsharp
for s in series do s.marker.colorscale <- "Viridis"

series
|> Chart.Plot
|> Chart.WithTitle "Viridis scale"
|> display

for s in series do s.marker.colorscale <- "Hot"

series
|> Chart.Plot
|> Chart.WithTitle "Hot scale"
|> display

for s in series do s.marker.colorscale <- "Jet"

series
|> Chart.Plot
|> Chart.WithTitle "Jet scale"
|> display
```

# Rendering Histograms
Let's have a look at using histograms, the next cell sets up some generators.

```fsharp
let count = 20
let dates = [for d in 1 .. count -> DateTime.Now.AddMinutes(float(generator.Next(d, d + 30)))]
```

Now let's define histogram traces:

```fsharp
let openByTime =
    Histogram(
        x = dates,
        y = [for y in 1 .. count -> generator.Next(0, 200)],
        name = "Open")
    
let closeByTime =
    Histogram(
        x = dates,
        y = [for y in 1 .. count -> generator.Next(0, 200)],
        name = "Close")
    
[openByTime; closeByTime]
|> Chart.Plot
```

The Histogram generator will automatically count the number of items per bin. 

Setting `histfunc` to `"sum"` we can now add up all the values contained in each bin.
Note that we are creatng bin using the `x` data point and we are using bydefault autobinx

```fsharp
let openByTime =
    Histogram(
        x = dates,
        y = [for y in 1 .. count -> generator.Next(0, 200)],
        name = "Open",
        histfunc = "sum")
    
let closeByTime =
    Histogram(
        x = dates,
        y = [for y in 1 .. count -> generator.Next(0, 200)],
        name = "Close",
        histfunc = "sum")
    
[openByTime; closeByTime]
|> Chart.Plot
```

# Area chart and Polar Area chart


By populating hte property `fill` of a `Scatter` trace the chart will render as area chart.

Here is set to `"tozeroy"` which will create a fill zone underneath the line reachin to the 0 of the y axis.

```fsharp
let openSeries =
    Scatter(
        name = "Open",
        x = [1; 2; 3; 4],
        y = [10; 15; 13; 17],
        fill = "tozeroy",
        mode= "lines")
    
let closeSeries =
    Scatter(
        name = "Close",
        x = [1; 2; 3; 4],
        y = [3; 5; 11; 9],
        fill = "tozeroy",
        mode= "lines")

[openSeries; closeSeries]
|> Chart.Plot
|> Chart.WithTitle "Open vs Close"
```

With one `fill` set to `"tonexty"` the cahrt will fill the aread between traces.

```fsharp
openSeries.fill <- None
closeSeries.fill <- "tonexty"

[openSeries; closeSeries]
|> Chart.Plot
|> Chart.WithTitle "Open vs Close"
```

Using `Area` traces we can generate radial area chart. In this example we are using cardinal points to xpress angular values.
The list `["North"; "N-E"; "East"; "S-E"; "South"; "S-W"; "West"; "N-W"]` will be autoimatically translated to angular values.

```fsharp
let winDirections =  ["North"; "N-E"; "East"; "S-E"; "South"; "S-W"; "West"; "N-W"]

let areaTrace1 =
    Area(
        r = [77.5; 72.5; 70.0; 45.0; 22.5; 42.5; 40.0; 62.5],
        t = winDirections,
        name = "11-14 m/s",
        marker = Marker(color = "rgb(106,81,163)"))

let areaTrace2 =
    Area(
        r = [57.49999999999999; 50.0; 45.0; 35.0; 20.0; 22.5; 37.5; 55.00000000000001],
        t = winDirections,
        name = "8-11 m/s",
        marker = Marker(color = "rgb(158,154,200)"))

let areaTrace3 = 
    Area(
        r = [40.0; 30.0; 30.0; 35.0; 7.5; 7.5; 32.5; 40.0],
        t = winDirections,
        name = "5-8 m/s",
        marker = Marker(color = "rgb(203,201,226)"))

let areaTrace4 =
    Area(
        r = [20.0; 7.5; 15.0; 22.5; 2.5; 2.5; 12.5; 22.5],
        t = winDirections,
        name = "< 5 m/s",
        marker = Marker(color = "rgb(242,240,247)"))

let areaLayout =
    Layout(
        title = "Wind Speed Distribution in Laurel, NE",
        font = Font(size = 16.),
        legend = Legend(font = Font(size = 16.)),
        radialaxis = Radialaxis(ticksuffix = "%"),
        orientation = 270.)
    
[areaTrace1; areaTrace2; areaTrace3; areaTrace4]
|> Chart.Plot
|> Chart.WithLayout areaLayout
```

```fsharp

```
