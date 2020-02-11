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

# Displaying output in a .NET notebook


When writing C# in a .NET notebook, the C# scripting language is used, which you might be familiar with from using the C# Interactive window in Visual Studio. This dialect of C# allows you to end a code submission without a semicolon, which tells C# scripting to return the value of the expression.

```csharp
var x = "Hello!";
display(x);
x
```

When you end a cell with an expression evaluation like this, it is the return value of the cell. There can only be a single return value for a cell. If you add more code after a return value expression, you'll see a compile error.

There are also several ways to display information without using the return value. The most intuitive one for many .NET users is to write to the console:

```csharp
Console.Write("hello");
Console.Write(" ");
Console.WriteLine("world");
```

But a more familiar API for many notebook users would be the `display` method.

```csharp
display("hello");
display("world");
```

Each call to `display` writes an additional display data value to the notebook.
You can also update an existing displayed value by calling `Update` on the object returned by a call to `display`.

```csharp
var fruitOutput = display("Let's get some fruit!");
var basket = new [] {"apple", "orange", "coconut", "pear", "peach"};

foreach (var fruit in basket)
{
    System.Threading.Thread.Sleep(1000);
    fruitOutput.Update($"I have 1 {fruit}.");    
}

System.Threading.Thread.Sleep(1000);

fruitOutput.Update(basket);
```

---
**_See also_**
* [Object formatters](Object%20formatters.ipynb)
* [HTML](HTML.ipynb)

```csharp

```
