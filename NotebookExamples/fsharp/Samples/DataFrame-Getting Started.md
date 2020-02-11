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

This demonstrates the use of `Microsoft.Data.Analysis` data frames with F#.You can open this example online using [MyBinder](https://mybinder.org/v2/gh/mwouts/interactive/master?filepath=fsharp%2FSamples%2FDataFrame-Getting%20Started.ipynb).First, get the package:

```fsharp
#r "nuget: Microsoft.Data.Analysis"
open Microsoft.Data.Analysis
```

Create 3 columns to hold values of types `DateTime`, `int`, and `string`

```fsharp
let dateTimes = PrimitiveDataFrameColumn<DateTime>("DateTimes") // Default length is 0.
let ints = PrimitiveDataFrameColumn<int>("Ints", 3L) // Makes a column of Length 3. Filles with nulls initially.
let strings = StringDataFrameColumn("Strings", 3L)
```

Add some datetimes

```fsharp
dateTimes.Append(Nullable(DateTime.Parse("2019/01/01")))
dateTimes.Append(Nullable(DateTime.Parse("2019/01/01")))
dateTimes.Append(Nullable(DateTime.Parse("2019/01/02")))
```

Create a `DataFrame` with 3 columns

```fsharp
let df = DataFrame([dateTimes; ints; strings]: DataFrameColumn list)

```

```fsharp
df
```

Create a formatter

```fsharp
open System.IO

let register (df:DataFrame) (writer:TextWriter) =
    let headers = new ResizeArray<IHtmlContent> ()
    headers.Add(th.innerHTML(i.innerHTML("index")))
    headers.AddRange(df.Columns.Select(fun c -> (th.innerHTML(c.Name) :> IHtmlContent)))
    let rows = ResizeArray<ResizeArray<IHtmlContent>>()
    let take = 20
    for i in 0 .. (Math.Min(take, int(df.Rows.Count)) - 1) do
        let cells = ResizeArray<IHtmlContent>()
        cells.Add(td.innerHTML(i));
        for o in df.Rows.[int64(i)] do
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
df
```

Change a value directly through df

```fsharp
df.[0L, 1] <- 10
df
```

We can also modify the values in the columns through indexers defined in `PrimitiveDataColumn` and `StringColumn`

```fsharp
ints.[1L] <- Nullable 100
strings.[1L] <- "Foo!"
df
```

Check the data type

```fsharp
df.Info()
```

The `DataFrame` and the base `DataFrameColumn` class that all columns derive from expose a number of useful APIs: binary operations, computations, joins, merges, handling missing values and more.

```fsharp
df.["Ints"].Add(5, inPlace=true)
df
```

```fsharp
df.["Ints"] <- (ints / 5) * 100
df
```

Let's `null` it up!

```fsharp
df.["Ints"].FillNulls(-1, inPlace=true)
df.["Strings"].FillNulls("Bar", inPlace=true)
df
```

DataFrame exposes `Columns` property that we can enumerate over to access our columns. Here's how you can access the first row, though.

```fsharp
let row0 = df.Rows.[0L]
row0
```

```fsharp
open System.IO

let register (dataFrameRow:DataFrameRow) (writer:TextWriter) =
    let cells = ResizeArray<IHtmlContent>()
    cells.Add(td.innerHTML(i));
    for i in dataFrameRow do
            cells.Add(td.innerHTML(i))
    
    let t =
        table.innerHTML([|
            tbody.innerHTML(cells)
        |])

    writer.Write(t)

Formatter<DataFrameRow>.Register( (fun dataFrameRow writer -> register dataFrameRow writer), mimeType = "text/html")
```

```fsharp
row0
```

Let's take a look at `Filter`, `Sort`, and `GroupBy`.

```fsharp
// Sort our dataframe using the Ints column
df.Sort("Ints", ascending=true)
```

```fsharp
// GroupBy
let grouped = df.GroupBy("DateTimes")
// Count of values in each group
grouped.Count()
```

```fsharp
let intGroupSum = grouped.Sum("Ints");
intGroupSum
```

```fsharp
open XPlot.Plotly
open System.Linq
```

```fsharp
#r "nuget:MathNet.Numerics"
```

```fsharp
open MathNet.Numerics.Distributions
```

```fsharp
let mean = 0.0
let stdDev = 0.1

let normalDist = new Normal(mean, stdDev);
```

```fsharp
let doubles = PrimitiveDataFrameColumn<double>("Normal Distribution", normalDist.Samples().Take(1000));
// let ints = PrimitiveDataFrameColumn<int>("Ints", 3L) 
display(Chart.Plot(
    Graph.Histogram(x = doubles, nbinsx = 30)
));
```

```fsharp

```
