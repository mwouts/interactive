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

# Outputting HTML in a notebook


## Display Helpers

There are a number of helper methods for writing HTML that are available by default in a .NET notebook.


## HTML


If you want to write out a `string` as HTML, you can use the `HtmlString` object:

```fsharp
 display(HtmlString("<b style=\"color:blue\">Hello!</b>"))
```

Displaying HTML using a `string` directly will display the actual string rather than rendering it as HTML.

```fsharp
display("<b style=\"color:blue\">Hello!</b>")
```

## Javascript


You may also want to output JavaScript. You can do this using the `Javascript` helper.

```fsharp
Javascript(@"alert(""Hello!"");")
```

```fsharp

```
