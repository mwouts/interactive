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

```fsharp
#r "nuget:bl=true"
#r "nuget:Microsoft.ML,version=1.4.0"
#r "nuget:Microsoft.ML.AutoML,version=0.16.0"
#r "nuget:Microsoft.Data.Analysis,version=0.1.0"
    
open Microsoft.Data.Analysis
open XPlot.Plotly
open Microsoft.AspNetCore.Html
open System.IO
```

```fsharp
let register (df:DataFrame) (writer:TextWriter) =
    let headers = new ResizeArray<IHtmlContent> ()
    headers.Add(th.innerHTML(i.innerHTML("index")))
    headers.AddRange(df.Columns.Select(fun c -> (th.innerHTML(c.Name) :> IHtmlContent)))
    let rows = ResizeArray<ResizeArray<IHtmlContent>>()
    let take = 20
    for i in 0 .. (Math.Min(take, int(df.RowCount)) - 1) do
        let cells = ResizeArray<IHtmlContent>()
        cells.Add(td.innerHTML(i));
        for o in df.[int64(i)] do
            cells.Add(td.innerHTML(o))
        rows.Add(cells)
    
    let t =
        table.innerHTML([|
            thead.innerHTML(headers)
            tbody.innerHTML(rows.Select(fun r -> tr.innerHTML(r)))
        |])

    writer.Write(t)

Formatter<DataFrame>.Register( (fun df writer -> register df writer), mimeType = "text/html")
```

```fsharp
open System.Net.Http
let housingPath = "housing.csv"
if not(File.Exists(housingPath)) then
    let contents = HttpClient().GetStringAsync("https://raw.githubusercontent.com/ageron/handson-ml2/master/datasets/housing/housing.csv").Result
    File.WriteAllText("housing.csv", contents)
```

```fsharp
let housingData = DataFrame.LoadCsv(housingPath)
housingData
```

```fsharp
housingData.Description()
```

```fsharp
let graph =
    Histogram(x = housingData.["median_house_value"],
              nbinsx = 20)
graph |> Chart.Plot
```

```fsharp
let graph =
    Graph.Scattergl(
        x = housingData.["longitude"],
        y = housingData.["latitude"],
        mode = "markers",
        marker =
            Graph.Marker(
                color = housingData.["median_house_value"],
                colorscale = "Jet"))

let plot = Chart.Plot(graph)
plot.Width <- 600
plot.Height <- 600
display(plot)
```

```fsharp
let Shuffle (arr:int[]) =
    let rnd = Random()
    for i in 0 .. arr.Length - 1 do
        let r = i + rnd.Next(arr.Length - i)
        let temp = arr.[r]
        arr.[r] <- arr.[i]
        arr.[i] <- temp
    arr

let randomIndices = (Shuffle(Enumerable.Range(0, (int (housingData.RowCount))).ToArray()))

let testSize = int (float (housingData.RowCount) * 0.1)
let trainRows = randomIndices.[testSize..]
let testRows = randomIndices.[..testSize - 1]

let housing_train = housingData.[trainRows]
let housing_test = housingData.[testRows]

display(housing_train.RowCount)
display(housing_test.RowCount)
```

```fsharp
#!time

open Microsoft.ML
open Microsoft.ML.Data
open Microsoft.ML.AutoML

let mlContext = MLContext()

let experiment = mlContext.Auto().CreateRegressionExperiment(maxExperimentTimeInSeconds = 15u)
let result = experiment.Execute(housing_train, labelColumnName = "median_house_value")
```

```fsharp
type RunDetails = System.Collections.Generic.IEnumerable<RunDetail<RegressionMetrics>>
let scatters =
    result.RunDetails
        .Where(fun d -> not (isNull d.ValidationMetrics))
        .GroupBy(
            (fun r -> r.TrainerName),
            (fun (name:string) (details:RunDetails) -> 
                Graph.Scattergl(
                    name = name,
                    x = details.Select(fun r -> r.RuntimeInSeconds),
                    y = details.Select(fun r -> r.ValidationMetrics.MeanAbsoluteError),
                    mode = "markers",
                    marker = Graph.Marker(size = 12))))

let chart = Chart.Plot(scatters)
chart.WithXTitle("Training Time")
chart.WithYTitle("Error")
display(chart)

Console.WriteLine("Best Trainer:{0}", result.BestRun.TrainerName);
```

```fsharp
let testResults = result.BestRun.Model.Transform(housing_test)

let trueValues = testResults.GetColumn<float32>("median_house_value")
let predictedValues = testResults.GetColumn<float32>("Score")

let predictedVsTrue =
    Graph.Scattergl(
        x = trueValues,
        y = predictedValues,
        mode = "markers")

let maximumValue = Math.Max(trueValues.Max(), predictedValues.Max())

let perfectLine =
    Graph.Scattergl(
        x = [| 0.0f; maximumValue |],
        y = [| 0.0f; maximumValue |],
        mode = "lines")

let chart = Chart.Plot([| predictedVsTrue; perfectLine |])
chart.WithXTitle("True Values")
chart.WithYTitle("Predicted Values")
chart.WithLegend(false)
chart.Width = 600
chart.Height = 600
display(chart)
```

```fsharp

```
