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

# Outputting HTML in a notebook


## Display Helpers

There are a number of helper methods for writing HTML that are available by default in a .NET notebook.


### HTML


If you want to write out a `string` as HTML, you can use the `HTML` method:

```csharp
display(HTML("<b style=\"color:blue\">Hello!</b>"));
```

Displaying HTML using a `string` directly will display the actual string rather than rendering it as HTML. 

```csharp
display("<b style=\"color:blue\">Hello!</b>");
```

The `HTML` method signals that the content is HTML because its return type, `HtmlString`, implements `IHtmlContent`:

```csharp
var someHtml = HTML("<b style=\"color:blue\">Hello!</b>");

display(someHtml.GetType());
```

### Javascript


You may also want to output JavaScript. You can do this using the `Javascript` helper.

```csharp
Javascript(@"alert(""Hello!"");");
```

## PocketView (C#)


For more complex HTML, you can use the PocketView API. Lets start with an example:

```csharp
display(
    span(
        img[src:"https://en.wikipedia.org/favicon.ico", style:"height:1.5em"],
        a[href: @"https://en.wikipedia.org", target: "blank", style:"color:green"](b("Wikipedia"))
    )
);
```

PocketView is an API for concisely writing HTML, in the terminology of HTML, using C# code. Just like the `HTML` method, it returns an object that implements `IHtmlContent`. You can see the actual HTML code by converting a `PocketView` to a string:

```csharp
var pocketView = span(
        img[src:"https://en.wikipedia.org/favicon.ico", style:"height:1.5em"],
        a[href: @"https://en.wikipedia.org", target: "blank", style:"color:green"](b("Wikipedia")));

display(pocketView.ToString());
```

The PocketView API provides a number of top-level properties in your notebook that can be used to create various HTML tags. Here's the list of tags that are supported by default:

```csharp
var pocketViewTagMethods = typeof(PocketViewTags)
    .GetProperties()
    .Select(m => m.Name);

display(pocketViewTagMethods);
```

Each of these properties returns a `PocketView` instance that can then be filled in with some content by passing arguments to it like a method call.

```csharp
var pocketView = i("Hello!");

display(pocketView);
```

A `PocketView` instance can also be decorated with attributes using square brackets.

```csharp
var pocketView = span[style:"font-style:italic"]("Hello!");

display(pocketView);
```

You'll notice that if you pass a `string` to `PocketView`, it will be HTML encoded for you:

```csharp
PocketView pocketView = span("<div>This string looks like HTML but it will be HTML encoded.</div>");

display(pocketView);

display("Have a look at the actual HTML:");

display(pocketView.ToString());
```

If you don't want the content to be encoded, simply pass it wrapped in a type that implements `IHtmlContent`.

```csharp
var htmlContent = HTML("<i>This won't be HTML encoded.</i>");

PocketView pocketView = span(
    htmlContent
);

display(pocketView);
```

You can pass other types of objects of into a `PocketView` as well. When you do this, they're formatted using the plain text formatter, which by default expands the object's properties.

```csharp
PocketView pocketView = b(
    new { Fruit = "apple", Texture = "smooth" }
);

display(pocketView);
```

## Magic Commands

There are also several magic commands that can be used to output HTML in your .NET notebook.

You can output HTML...

```csharp
#!html

<b>Hello!</b>
```

...or run JavaScript...

```csharp
#!javascript

alert("hello");
```

...or render Markdown.

```csharp
#!markdown

Write a **list** ...
* first
* second

...or a _table_...

|Fruit    |Texture |
|---------|--------|
|apple    |smooth  |
|durian   |bumpy   |
```

---
**_See also_**
* [Object formatters](Object%20formatters.md)
* [Displaying output](Displaying%20output.md)

```csharp

```
