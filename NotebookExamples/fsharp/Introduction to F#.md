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

# Introduction to F# #

F# is an [open-source, cross-platform functional programming language](http://aka.ms/fsharphome) for .NET.

F# has features and idioms to support functional programming while also offering clean interop with C# and existing .NET codebases and systems. It can use anything in the [NuGet](https://www.nuget.org/) ecosystem, as this notebook will also demonstrate.


## F# basics

Let's start with some simple arithmetic:

```fsharp
(12/4 + 5 + 7) * 4 - 18
```

Arithmetic is nice, but there's so much more you can do. Here's how you can generate some data using the `[start .. end]` range syntax:

```fsharp
let numbers = [0 .. 10]
numbers
```

You can use slices with the `.[start .. end]` syntax to slice a subset of the data you just generated:

```fsharp
// Take the numbers from 2nd index to the 5th
numbers.[2..5]
```

And you can use indexer syntax (`.[index]`) to access a single value:

```fsharp
numbers.[3]
```

## Functions in F# #

Since F# is a functional language, functions are one of the first things to learn. You do that with the `let` keyword. F#, like Python, uses indentation to define code blocks:

```fsharp
let sampleFunction x =
    2*x*x - 5*x + 3
```

F# uses type inference to figure out types for you. But if needed, you can specify types explicitly:

```fsharp
let sampleFunction' (x: int) =
    2*x*x - 5*x + 3
```

When calling F# functions, parentheses are optional:

```fsharp
sampleFunction 5
```

```fsharp
sampleFunction' 12
```

You can define and compose F# functions easily:

```fsharp
let negate x = -x
let square x = x * x
let print x = printfn "The number is %d" x

let squareNegateThenPrint x =
    print (negate (square x))
    
squareNegateThenPrint 5
```

The pipeline operator `|>` is used extensively in F# code to chain functions and arguments together. It helps readability when building functional "pipelines":

```fsharp
// Redefine the function with pipelines
let squareNegateThenPrint x =
    x
    |> square
    |> negate
    |> print

squareNegateThenPrint 5
```

## Strings, tuples, lists, and arrays

Strings in F# use `"` quotations. You can concatenate them with the `+` operator:

```fsharp
let s1 = "Hello"
let s2 = "World"

s1 + ", " + s2 + "!"
```

You can use triple-quoted strings (`"""`) if you want to have a string that contains quotes:

```fsharp
"""A triple-quoted string can contain quotes "like this" anywhere within it"""
```

### Tuples

Tuples are simple combinations of data items into a single value. The following defines a tuple of an integer, string, and double:

```fsharp
(1, "fred", Math.PI)
```

You can also create `struct` tuples when you have performance-sensitive environments:

```fsharp
struct (1, Math.PI)
```

### Lists

Lists are linear sequences of values of the same type. In fact, you've already seen them above when we generated some numbers!

```fsharp
[0 .. 10]
```

You can use list comprehensions to generate more advanced data programmatically:

```fsharp
let thisYear = DateTime.Now.Year

let fridays =
    [
        for month in 1 .. 10 do
            for day in 1 .. DateTime.DaysInMonth(thisYear, month) do
                let date = DateTime(thisYear, month, day)
                if date.DayOfWeek = DayOfWeek.Friday then
                    date.ToShortDateString()
    ]

// Get the first five fridays of this year
fridays
|> List.take 5
```

Since you can slice lists, the first five fridays could also be done like this:

```fsharp
fridays.[..4]
```

### Arrays

Arrays are very similar to lists. A key difference is that array internals are mutable. They also have better performance characteristics than lists.

```fsharp
let firstTwoHundred = [| 1 .. 200 |]
firstTwoHundred.[197..]
```

Processing lists and arrays is typically done by built-in and custom functions:

```fsharp
// Filter the previous list of numbers and sum their squares.
firstTwoHundred
|> Array.filter (fun x -> x % 3 = 0)
|> Array.sumBy (fun x -> x * x)
```

## Types

Although F# is succinct, it actually uses static typing! Types are central to F# programming, especially when you want to model more complicated data to manipulate later in a program.


### Records

Record types are used to combine different kinds of data into an aggregate. They cannot be `null` and come with default comparison and equality.

```fsharp
type ContactCard =
    { Name: string
      Phone: string
      ZipCode: string }

// Create a new record
{ Name = "Alf"; Phone = "(555) 555-5555"; ZipCode = "90210" }
```

In this notebook environment, records print with a table-like output by default.

You can access record labels with `.`-notation:

```fsharp
let alf = { Name = "Alf"; Phone = "(555) 555-5555"; ZipCode = "90210" }
alf.Phone
```

Records are comparable and equatable:

```fsharp
// Create another record
let ralph = { Name = "Ralph"; Phone = "(123) 456-7890"; ZipCode = "90210" }

// Check if they're equal
alf = ralph
```

You'll find yourself writing functions that operate on records all the time:

```fsharp
let showContactCard contact =
    contact.Name + " - Phone: " + contact.Phone + ", Zip: " + contact.ZipCode
    
showContactCard alf
```

### Discriminated Unions

Discriminated Unions (often called DUs) provide support for values that can be one of a number of named cases. These cases can be completely different from one another.

In the following example, we combine records with a discriminated union:

```fsharp
type Shape =
    | Rectangle of width: float * length: float
    | Circle of radius: float
    | Prism of width: float * height: float * faces: int
    
let rect = Rectangle(length = 1.3, width = 10.0)
let circ = Circle (1.0)
let prism = Prism(width = 5.0, height = 2.0, faces = 3)
        
prism
```

## Pattern matching

The best way to work with DUs is pattern matching. Using the previously-defined type definitions, we can model getting the height of a shape.

```fsharp
let height shape =
    match shape with
    | Rectangle(width = h) -> h
    | Circle(radius = r) -> 2.0 * r
    | Prism(height = h) -> h
    
let rectHeight = height rect
let circHeight = height circ
let prismHeight = height prism

printfn "rect is %0.1f, circ is %0.1f, and prism is %0.1f" rectHeight circHeight prismHeight
```

You can pattern match on more than just discriminated unions. Here we write a recursive function with `rec` to process lists:

```fsharp
// See if x is a multiple of n
let isPrimeMultiple n x =
    x = n || x % n <> 0
    
// Process lists recursively.
// '[]' means the empty list.
// 'head' is an item in the list.
// 'tail' is the rest of the list after 'head'.
let rec removeMultiples ns xs =
    match ns with
    | [] -> xs
    | head :: tail ->
        xs
        |> List.filter (isPrimeMultiple head)
        |> removeMultiples tail
        
let getPrimesUpTo n =
    let max = int (sqrt (float n))
    removeMultiples [2 .. max] [1 .. n]
    
// Primes up to 25
getPrimesUpTo 25
```

## Options

A built-in DU type is the F# option type. It is used prominently in F# code. Options can either be `Some` or `None`, and they're best used when you want to account for when there may not be a value.

```fsharp
let keepIfPositive a =
    if a > 0 then
        Some a
    else 
        None
        
keepIfPositive 12
```

Options are often used when searching for values. Here's how you can incorporate them into list processing:

```fsharp
let rec tryFindMatch predicate lst =
    match lst with
    | [] -> None
    | head :: tail ->
        if predicate head then
            Some head
        else
            tryFindMatch predicate tail
          
let greaterThan100 x = x > 100

tryFindMatch greaterThan100 [25; 50; 100; 150; 200]
```

## Parallel Programming

For more CPU-intensive tasks, you can take advantage of built-in parallelism:

```fsharp
#!time

let bigArray = [| 0 .. 100_000 |]

let rec fibonacci n = if n <= 2 then n else fibonacci (n-1) + fibonacci (n-2)

// We'll use the '%A' print formatter for F# constructs for these results, since they are enormous
let results =
    bigArray
    |> Array.Parallel.map (fun n -> fibonacci (n % 25))

printfn "%A" results
```

Because F# functions are first-class values, you can trivially do things like initialize expensive functions in parallel with the `Array.Parallel` module. This is quite common in numerics-intensive F# code.

Here's an example where you can compute as many fibonacci numbers as there are threads in your current process. The `#!time` magic command shows the wall-clock time it took to perform the operation:

```fsharp
#!time

// Restrict the number of threads to a max of 25
let nThreads = min 25 Environment.ProcessorCount
    
Array.Parallel.init nThreads fibonacci
```

It's also worth noting how much faster the second cell ran than the first one. This is because it doesn't use call `printfn` with the `%A` formatter. Although this kind of formatting is very convenient in F#, it comes at a performance cost!


## Learn more

There are a lot of learning resources for F# that go far beyond this notebook.

Check out the [F# docs homepage](https://docs.microsoft.com/dotnet/fsharp/) for an organized set of learning material.

To learn more about using F# with this Jupyter kernel, we recommmend the following notebooks:

* [Displaying output](Docs/Displaying%20output.md)
* [Importing Packages](Docs/Importing%20packages.md)
* [Plotting with Xplot](Docs/Plotting%20with%20Xplot.md)
* [Object Formatters](Docs/Object%20Formatters.md)

For more advanced samples, check out the following:

* [HousingML](Samples/HousingML.md)
