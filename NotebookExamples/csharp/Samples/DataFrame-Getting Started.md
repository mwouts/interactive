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

```csharp
#r "nuget:Microsoft.Data.Analysis,0.2.0"
using Microsoft.Data.Analysis;
```

```csharp
PrimitiveDataFrameColumn<DateTime> dateTimes = new PrimitiveDataFrameColumn<DateTime>("DateTimes"); // Default length is 0.
PrimitiveDataFrameColumn<int> ints = new PrimitiveDataFrameColumn<int>("Ints", 3); // Makes a column of length 3. Filled with nulls initially
StringDataFrameColumn strings = new StringDataFrameColumn("Strings", 3); // Makes a column of length 3. Filled with nulls initially
```

```csharp
// Append 3 values to dateTimes
dateTimes.Append(DateTime.Parse("2019/01/01"));
dateTimes.Append(DateTime.Parse("2019/01/01"));
dateTimes.Append(DateTime.Parse("2019/01/02"));
```

```csharp
DataFrame df = new DataFrame(dateTimes, ints, strings ); // This will throw if the columns are of different lengths
```

```csharp
df
```

```csharp
using Microsoft.AspNetCore.Html;
Formatter<DataFrame>.Register((df, writer) =>
{
    var headers = new List<IHtmlContent>();
    headers.Add(th(i("index")));
    headers.AddRange(df.Columns.Select(c => (IHtmlContent) th(c.Name)));
    var rows = new List<List<IHtmlContent>>();
    var take = 20;
    for (var i = 0; i < Math.Min(take, df.Rows.Count); i++)
    {
        var cells = new List<IHtmlContent>();
        cells.Add(td(i));
        foreach (var obj in df.Rows[i])
        {
            cells.Add(td(obj));
        }
        rows.Add(cells);
    }
    
    var t = table(
        thead(
            headers),
        tbody(
            rows.Select(
                r => tr(r))));
    
    writer.Write(t);
}, "text/html");
```

```csharp
df
```

```csharp
// To change a value directly through df
df[0, 1] = 10; // 0 is the rowIndex, and 1 is the columnIndex. This sets the 0th value in the Ints columns to 10
df
```

```csharp
// Modify ints and strings columns by indexing
ints[1] = 100;
strings[1] = "Foo!";
df
```

```csharp
// Indexing can throw when types don't match.
// ints[1] = "this will throw because I am a string";  
// Info can be used to figure out the type of data in a column. 
df.Info()
```

```csharp
// Add 5 to ints through the DataFrame
df["Ints"].Add(5, inPlace: true);
df
```

```csharp
// We can also use binary operators. Binary operators produce a copy, so assign it back to our Ints column 
df["Ints"] = (ints / 5) * 100;
df
```

```csharp
// Fill nulls in our columns, if any. Ints[2], Strings[0] and Strings[1] are null
df["Ints"].FillNulls(-1, inPlace: true);
df["Strings"].FillNulls("Bar", inPlace: true);
df
```

```csharp
// To inspect the first row
DataFrameRow row0 = df.Rows[0];
row0
```

```csharp
using Microsoft.AspNetCore.Html;
Formatter<DataFrameRow>.Register((dataFrameRow, writer) =>
{
    var cells = new List<IHtmlContent>();
    cells.Add(td(i));
    foreach (var obj in dataFrameRow)
    {
        cells.Add(td(obj));
    }
    
    var t = table(
        tbody(
            cells));
    
    writer.Write(t);
}, "text/html");
```

```csharp
row0
```

```csharp
// Filter rows based on equality
PrimitiveDataFrameColumn<bool> boolFilter = df["Strings"].ElementwiseEquals("Bar");
boolFilter
```

```csharp
DataFrame filtered = df.Filter(boolFilter);
filtered
```

```csharp
// Sort our dataframe using the Ints column
DataFrame sorted = df.Sort("Ints");
sorted
```

```csharp
// GroupBy 
GroupBy groupBy = df.GroupBy("DateTimes");
// Count of values in each group
DataFrame groupCounts = groupBy.Count();
groupCounts
```

```csharp
// Alternatively find the sum of the values in each group in Ints
DataFrame intsGroupSum = groupBy.Sum("Ints");
intsGroupSum
```

```csharp
using XPlot.Plotly;
using System.Linq;
```

```csharp
#r "nuget:MathNet.Numerics,4.9.0"
```

```csharp
using MathNet.Numerics.Distributions;
double mean = 0;
double stdDev = 0.1;

MathNet.Numerics.Distributions.Normal normalDist = new Normal(mean, stdDev);
```

```csharp
PrimitiveDataFrameColumn<double> doubles = new PrimitiveDataFrameColumn<double>("Normal Distribution", normalDist.Samples().Take(1000));
display(Chart.Plot(
    new Graph.Histogram()
    {
        x = doubles,
        nbinsx = 30
    }
));

```
