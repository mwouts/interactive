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

# Importing packages, libraries, and scripts

<!-- #region -->
You can load packages into a .NET notebook from NuGet using the following syntax:

```fsharp
#r "nuget:<package name>[,<version=package version>]"
```

If you don't provide an explicit package version, the latest available non-preview version will be loaded.

Here's an example:
<!-- #endregion -->

```fsharp
#r "nuget:FSharp.Data"
```

Now that the package is loaded, we can add some `using` statements and write some code.

```fsharp
open FSharp.Data

[<Literal>]
let url = "https://en.wikipedia.org/wiki/2017_Formula_One_World_Championship"

type F1_2017 = HtmlProvider<url>
    
let f1Calendar = F1_2017.Load(url).Tables.``Season calendar``

f1Calendar.Rows
|> Seq.map (fun x -> x.Circuit, x.Date)
```

<!-- #region -->
If you want to load an assembly that's already on disk, you can do so using this syntax:

```fsharp
#r "<path to .dll>"
```
<!-- #endregion -->

<!-- #region -->
You can load an F# script (typically a `.fsx` file) into the notebook using this syntax:

```fsharp
#load "<path to .fsx file>"
```
<!-- #endregion -->

```fsharp
// Example:
#load "some-fsharp-script-file.fsx"
```

```fsharp

```
