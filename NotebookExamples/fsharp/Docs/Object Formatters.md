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

# Object formatters


## Default formatting behaviors


When you return a value or a display a value in a .NET notebook, the default formatting behavior is to try to provide some useful information about the object. If it's an array or other type implementing `IEnumerable`, that might look like this:

```fsharp
display ["hello"; "world"]

Enumerable.Range(1, 5)
```

As you can see, the same basic structure is used whether you pass the object to the `display` method or return it as the cell's value.

Similarly to the behavior for `IEnumerable` objects, you'll also see table output for dictionaries, but for each value in the dictionary, the key is provided rather than the index within the collection.

```fsharp
// Cannot simply use 'dict' here, see https://github.com/dotnet/interactive/issues/12

let d = dict [("zero", 0); ("one", 1); ("two", 2)]
System.Collections.Generic.Dictionary<string, int>(d)
```

The default formatting behavior for other types of objects is to produce a table showing their properties and the values of those properties.

```fsharp
type Person = { FirstName: string; LastName: string; Age: int }

// Evaluate a new person
{ FirstName = "Mitch"; LastName = "Buchannon"; Age = 42 }
```

When you have a collection of such objects, you can see the values listed for each item in the collection:

```fsharp
let people =
    [
        { FirstName = "Mitch"; LastName = "Buchannon"; Age = 42 }
        { FirstName = "Hobie "; LastName = "Buchannon"; Age = 23 }
        { FirstName = "Summer"; LastName = "Quinn"; Age = 25 }
        { FirstName = "C.J."; LastName = "Parker"; Age = 23 }
    ]

people
```

Now let's try something a bit more complex. Let's look at a graph of objects. 

We'll redefine the `Person` class to allow a reference to a collection of other `Person` instances.

```fsharp
type Person =
    { FirstName: string
      LastName: string
      Age: int
      Friends: ResizeArray<Person> }

let mitch = { FirstName = "Mitch"; LastName = "Buchannon"; Age = 42; Friends = ResizeArray() }
let hobie = { FirstName = "Hobie "; LastName = "Buchannon"; Age = 23; Friends = ResizeArray() }
let summer =  { FirstName = "Summer"; LastName = "Quinn"; Age = 25; Friends = ResizeArray() }

mitch.Friends.AddRange([ hobie; summer ])
hobie.Friends.AddRange([ mitch; summer ])
summer.Friends.AddRange([ mitch; hobie ])

let people = [ mitch; hobie; summer ]
display people
```

That's a bit hard to read, right? 

The defaut formatting behaviors are thorough, but that doesn't always mean they're as useful as they might be. In order to give you more control in these kinds of cases, the object formatters can be customized from within the .NET notebook.


## Custom formatters


Let's clean up the output above by customizing the formatter for the `Person.Friends` property, which is creating a lot of noise. 

The way to do this is to use the `Formatter` API. This API lets you customize the formatting for a specific type. Since `Person.Friends` is of type `ResizeArray<Person>`, we can register a custom formatter for that type to change the output. Let's just list their first names:

```fsharp
Formatter<ResizeArray<Person>>.Register(
    fun people writer ->
        for person in people do
            writer.Write("person")
    , mimeType = "text/plain")

people
```

You might have noticed that `people` is of type `ResizeArray<Person>`, but the table output still includes columns for `LastName`, `Age`, and `Friends`. What's going on here?

Notice that the custom formatter we just registered was registered for the mime type `"text/plain"`. The top-level formatter that's used when we call `display` requests output of mime type `"text/html"` and the nested objects are formatted using `"text/plain"`. It's the nested objects, not the top-level HTML table, that's using the custom formatter here.

With that in mind, we can make it even more concise by registering a formatter for `Person`:

```fsharp
Formatter<Person>.Register(
    fun person writer ->
        writer.Write(person.FirstName)
    , mimeType = "text/plain");

people
```

Of course, you might not want table output. To replace the default HTML table view, you can register a formatter for the `"text/html"` mime type. Let's do that, and write some HTML using PocketView.

```fsharp

```
