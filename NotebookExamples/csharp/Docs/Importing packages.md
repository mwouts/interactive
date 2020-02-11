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

# Importing packages, libraries, and scripts

<!-- #region -->
You can load packages into a .NET notebook from NuGet using the following syntax:

```csharp
#r "nuget:<package name>[,<package version>]"
```

If you don't provide an explicit package version, the latest available non-preview version will be loaded.

Here's an example:
<!-- #endregion -->

```csharp
#r "nuget:System.Reactive.Linq, 4.1.5"
```

Now that the package is loaded, we can add some `using` statements and write some code.

```csharp
using System.Reactive;
using System.Reactive.Linq;
using System.Reactive.Concurrency;

var output = display("Counting...");

var sub = Observable
    .Interval(TimeSpan.FromSeconds(.5), CurrentThreadScheduler.Instance)    
    .Take(10) 
    .Subscribe(i => output.Update(i));
```

<!-- #region -->
If you want to load an assembly that's already on disk, you can do so using this syntax:

```csharp
#r "<path to .dll>"
```
<!-- #endregion -->

<!-- #region -->
You can load a C# script (typically a `.csx` file) into the notebook using this syntax:

```csharp
#load "<path to .csx file>"
```
<!-- #endregion -->

```csharp
#load "something.csx"
```

```csharp

```
