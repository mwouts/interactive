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

# Displaying output in a .NET (F#) notebook


When writing F# in a .NET notebook, you can write code similar to how you would with the F# Interactive tool.

```fsharp
1 + 3
```

The last value in a cell has its output displayed.

```fsharp
let r = System.Random()
r.Next(0,10)
```

```fsharp
"Hello, world!"
```

When you end a cell with an expression evaluation like this, it is the return value of the cell. There can only be a single return value for a cell. If you add more code after a return value expression, you'll see a compile error.

There are also several ways to display information without using the return value. The most intuitive one for many .NET users is to write to the console:

```fsharp
System.Console.WriteLine("Hello, world!")
printfn "Hello, world!"
```

But a more familiar API for many notebook users would be the `display` method.

```fsharp
display("Hello, world!")
```

Each call to `display` writes an additional display data value to the notebook.
You can also update an existing displayed value by calling `Update` on the object returned by a call to `display`.

```fsharp
let fruitOutput = display("Let's get some fruit!");
let basket = [| "apple"; "orange"; "coconut"; "pear"; "peach" |]

for fruit in basket do
    System.Threading.Thread.Sleep(1000);
    let updateMessage = sprintf "I have 1 %s" fruit
    fruitOutput.Update(updateMessage)

System.Threading.Thread.Sleep(1000);

fruitOutput.Update(basket);
```
