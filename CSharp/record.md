[with expression - C# reference | Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/with-expression)

[Explaining C# Records with Examples - Programmingempire](https://www.programmingempire.com/explaining-c-records-with-examples/#:~:text=Record%20is%20a%20construct%20in%20C%23%20that%20allows,behave%20like%20values%2C%20they%20are%20not%20value%20types.)

Record is a construct in C# that allows users to create objects. However, records are immutatble and behave like a value. Although records behave like values, they are not value tyhpes. 

In fact, records are a reference type. It must be remmebered that for records the content matters. In other words, we compare records by their content, also we can still make records mutable, but records are better suited for creating immutable types. 

[Use record types - C# tutorial | Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/tutorials/records)


Currently records can be defined as reference or value types based on the presence of the `struct` keyword when defining the record. In the snippet below `PersonReference` will be defined as a class (reference type) under the hood and `PersonStruct` will be defined as a value type (just like standard structs). When I refer to the term "under the hood" in the context of records I'm referring to the fact that the C# compiler performs lowering that transform our records into fully formed classes or structs based on how they're defined.

```csharp
public record PersonReference(string FirstName, string LastName, int Age);

public record struct PersonStruct(string FirstName, string LastName, int Age);
```

## Reasons to use Records 

#### Immutability By Default 

```csharp
public record Person(string FirstName, string LastName, int Age);

Person me = new("Aaron", "Bos", 30);

// Once instantiated the properties of me can't be changed
me.Age = 25; // Results in compilation error
```

In the code above, when trying to change `me.Age` the compiler will give the following error because the positional record defines its properties as `init-only`.

At this point you may be wondering how to handle a case when a method receives a record as an input and would like to modify it. This is where `with` expressions come in! When using `with` expressions we can produce a copy of an existing record while also modifying property values for the copied result in the process. Here's an example of a `with` expression that creates a copy of a record instance `me` and modifies the `Age` property to create a new record instance.

```csharp
public record Person(string FirstName, string LastName, int Age);

Person me = new("Aaron", "Bos", 30);

var youngerMe = me with { Age = 25 };
```

#### Value Equality

If you've worked with C# long enough, you'll understand that creating an instance of a class will create a reference type object. If we want to compare two instances of the same classes, we'll be comparing the reference locations in memory and not the contents of the objects themselves. There are some work arounds with implementing `IEquatable<T>` and overriding certain members of the base `object` type like `Equals` and `GetHashCode`. If value like equality is what we're looking for, then C# records are what we want. Below is an example highlighting the difference when determining equality of classes and records.


```csharp
public class PersonClass
{
  public string FirstName { get; set; }
  public string LastName { get; set; }
  public int Age { get; set; }
}

var meClass = new PersonClass { FirstName = "Aaron", LastName = "Bos", Age = 30 };
var otherMeClass = new PersonClass { FirstName = "Aaron", LastName = "Bos", Age = 30 };

// This prints false because the objects are different references
Console.WriteLine(meClass == otherMeClass);

public record Person(string FirstName, string LastName, int Age);

var me = new Person("Aaron", "Bos", 30);
var otherMe = new Person("Aaron", "Bos", 30);

// This prints true because records compare property values, not reference locations
Console.WriteLine(me == otherMe);
```

#### Records Are Concise 

The final sticking point for records, in my opinion, is how concise they are. In the majority of use cases our records can be defined in a single line of code. If we were to use a class to replace the record, it would require a little bit more screen real estate to do so. Let's continue using our `Person` example.

```csharp
public class PersonClass
{
  public string FirstName { get; set; }
  public string LastName { get; set; }
  public int Age { get; set; }
}

public record Person(string FirstName, string LastName, int Age);
```

