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

# Object formatters


## Default formatting behaviors


When you return a value or a display a value in a .NET notebook, the default formatting behavior is to try to provide some useful information about the object. If it's an array or other type implementing `IEnumerable`, that might look like this:

```csharp
display(new [] {"hello", "world"} );

Enumerable.Range(1, 5)
```

As you can see, the same basic structure is used whether you pass the object to the `display` method or return it as the cell's value.

Similarly to the behavior for `IEnumerable` objects, you'll also see table output for dictionaries, but for each value in the dictionary, the key is provided rather than the index within the collection.

```csharp
var dictionary = new Dictionary<string, int>
{
  ["zero"] = 0,
  ["one"] = 1,
  ["two"] = 2
};
dictionary
```

The default formatting behavior for other types of objects is to produce a table showing their properties and the values of those properties.

```csharp
class Person 
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public int Age { get; set; } 
}

display(new Person { FirstName = "Mitch", LastName = "Buchannon", Age = 42} );
```

When you have a collection of such objects, you can see the values listed for each item in the collection:

```csharp
var groupOfPeople = new [] 
{
    new Person { FirstName = "Mitch", LastName = "Buchannon", Age = 42 },
    new Person { FirstName = "Hobie ", LastName = "Buchannon", Age = 23 },
    new Person { FirstName = "Summer", LastName = "Quinn", Age = 25 },
    new Person { FirstName = "C.J.", LastName = "Parker", Age = 23 },
};

display(groupOfPeople);
```

Again, displaying the dictionary will show the items by key rather than index.

```csharp
display(groupOfPeople.ToDictionary(p => $"{p.FirstName}"));
```

Now let's try something a bit more complex. Let's look at a graph of objects. 

We'll redefine the `Person` class to allow a reference to a collection of other `Person` instances.

```csharp
class Person 
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public int Age { get; set; } 
    public List<Person> Friends { get; } = new List<Person>();
}


var mitch = new Person { FirstName = "Mitch", LastName = "Buchannon", Age = 42 };
var hobie = new Person { FirstName = "Hobie ", LastName = "Buchannon", Age = 23 };
var summer = new Person { FirstName = "Summer", LastName = "Quinn", Age = 25 };
var cj = new Person { FirstName = "C.J.", LastName = "Parker", Age = 23 };

mitch.Friends.AddRange(new [] { hobie, summer, cj });
hobie.Friends.AddRange(new [] { mitch, summer, cj });
summer.Friends.AddRange(new [] { mitch, hobie, cj });
cj.Friends.AddRange(new [] { mitch, hobie, summer });

var groupOfPeople = new List<Person> { mitch, hobie, summer, cj };

display(groupOfPeople);
```

That's a bit hard to read, right? 

The defaut formatting behaviors are thorough, but that doesn't always mean they're as useful as they might be. In order to give you more control in these kinds of cases, the object formatters can be customized from within the .NET notebook.


## Custom formatters


Let's clean up the output above by customizing the formatter for the `Person.Friends` property, which is creating a lot of noise. 

The way to do this is to use the `Formatter` API. This API lets you customize the formatting for a specific type. Since `Person.Friends` is of type `List<Person>`, we can register a custom formatter for that type to change the output. Let's just list their first names:

```csharp
Formatter<List<Person>>.Register((people, writer) => {
   foreach (var person in people)
   {
       writer.Write("person");
   }
}, mimeType: "text/plain");

groupOfPeople
```

You might have noticed that `groupOfPeople` is of type `List<Person>`, but the table output still includes columns for `LastName`, `Age`, and `Friends`. What's going on here?

Notice that the custom formatter we just registered was registered for the mime type `"text/plain"`. The top-level formatter that's used when we call `display` requests output of mime type `"text/html"` and the nested objects are formatted using `"text/plain"`. It's the nested objects, not the top-level HTML table, that's using the custom formatter here.

With that in mind, we can make it even more concise by registering a formatter for `Person`:

```csharp
Formatter<Person>.Register((person, writer) => {
   writer.Write(person.FirstName);
}, mimeType: "text/plain");

groupOfPeople
```

Of course, you might not want table output. To replace the default HTML table view, you can register a formatter for the `"text/html"` mime type. Let's do that, and write some HTML using PocketView.

```csharp
Formatter<List<Person>>.Register((people, writer) => 
{
    foreach (var person in people)
    {
        writer.Write(
            span(
                b(person), 
                " ",
                i($"({person.Age} years old and has {person.Friends.Count} friends)"),
                br));
    }
}, mimeType: "text/html");

groupOfPeople
```

---
**_See also_**
* [Displaying output](Displaying%20output.ipynb)
* [HTML](HTML.ipynb)

```csharp

```
