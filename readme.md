---
title: 'Exploring C# 8 New Language Features'
---

# Exploring C# 8 New Language Features

## Welcome

### AuthorFirst AuthorLast

* ☁️ Qualification
* ☁️ Qualification

<https://social-media.link>

### Agenda

* [C# Versions](#c-versions)
* [C# Member Enhancements](#c-member-enhancements)
* [Pattern Matching](#pattern-matching)
* [C# Expression Enhancements](#c-expression-enhancements)
* [C# Futures](#c-futures)

## C# Versions

### C# beginnings

* Originally released as part of **Visual Studio .NET 2002**
* Was very similar to Java in syntax
* Had the *basic build blocks* of an [object-oriented programming language](https://wikipedia.org/wiki/Object-oriented_programming)

### C# 2 - 4

* Introduced more advanced features
  * Generics
  * Anonymous members and types
  * Nullable types
  * Iterators
  * Lambda Expressions
  * Dynamic
  * Interop

### C# 5

* Introduced language-native asynchronous keywords (``async``/``await``)
* Introduced Task-based asynchronous programming

### C# 6

* Added many quality-of-life features
  * Advanced exception filter syntax
  * Auto-initialized properties
  * Members with expression bodies
  * String interpolation
  * Null conditionals
* Released side-by-side with Visual Studio 2015
* **Roslyn** meant that C# was now compiled using C#
  * The rate of progress can now grow exponentially
* 👉 Many C# developers are **here**

### C# 7 & 8

* Out variables
* Tuples
* Discards
* Local functions
* Async Main methods
* Pattern matching
* Default interface implementations
* Using delcarations
* Async streams
* ...and more...

### Our Goals

* Cover many of the enhancements in C# 7 & 8
* Use these new enhanced features side-by-side with language features from C# 6 and before

## C# Member Enhancements

### Synchronous Main Entry Point

```csharp
static void Main(string[] args)
{
    Console.WriteLine("Hello World!");
}
```

::: notes

Static, synchronous method with arguments

:::

### Calling Async Methods from Main

```csharp
static void Main(string[] args)
{
    var task = new Library().DoSomething();

    task.Wait();

    var result = task.Result;
}
```

### Calling More Than One Async Method

```csharp
static void Main(string[] args)
{
    var taskOne = new Library().DoSomething();

    var taskTwo = new Library().DoSomethingElse();

    Task.WaitAll(taskOne, taskTwo);

    var resultOne = taskOne.Result;

    var resultTwo = taskTwo.Result;
}
```

### Refactor Async Calls To Another Method

```csharp
static void Main(string[] args)
{
    Console.WriteLine("Hello World!");
    AsyncCalls().Wait();
}

static async Task AsyncCalls()
{
    var resultOne = await new Library().DoSomething();

    var resultTwo = await new Library().DoSomethingElse();
}
```

### Async Main Entry Point

```csharp
static async Task Main(string[] args)
{
    Console.WriteLine("Hello World!");

    var resultOne = await new Library().DoSomething();

    var resultTwo = await new Library().DoSomethingElse();
}
```

### Async Main Advantages

* Many cloud SDKs/libraries are asynchronous by default
* Reduces the complexity of your **Program.cs** file

### ASP.NET Bootstrap - Synchronous (default)

```csharp
public static void Main(string[] args)
{
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        })
        .Build()
        .Run();
}
```

### ASP.NET Bootstrap - Asynchronous

```csharp
public static async Task Main(string[] args)
{
    await Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        })
        .Build()
        .RunAsync();
}
```

### Interfaces and Implementations

* Typically, must implement **all** members of an interface
* What if you wanted to ship a default implementation with the interface?
  * Useful if you need to add members to an in-use interface

### Example Interface

```csharp
public interface IPerson
{
    string FirstName { get; }

    string LastName { get; }
}
```

### Default Interface Member Implementations

```csharp
public interface IPerson
{
    string FirstName { get; }

    string LastName { get; }

    string GetFullName() {
        return $"{FirstName} {LastName}";
    }
}
```

### Example Interface Class Implementation

```csharp
public class User : IPerson
{
    public string FirstName { get; set; }

    public string LastName { get; set; }
}
```

### Usage

```csharp
IPerson me = new User { FirstName = "Sidney", LastName = "Andrews" };
Console.WriteLine(me.GetFullName());
```

### Local Functions

Methods declared within the context of another method
Ideal for units of logic only called from within the current method
Used commonly to separate parameter validation from method implementation

::: notes

<https://docs.microsoft.com/dotnet/csharp/whats-new/csharp-7#local-functions>

:::

### Example Method

```csharp
public string ConcatenateNumbers(params int[] values)
{
    if (values.Count() == 0)
        throw new ArgumentException(nameof(values));

    return $"[{String.Join(", ", values)}]";
}
```

### Example Local Functions

```csharp
public string ConcatenateNumbers(params int[] values)
{
    if (!CheckNumbers())
        throw new ArgumentException(nameof(values));

    return CombineNumbers();

    string CombineNumbers()
    {
        return $"[{String.Join(", ", values)}]";
    }

    bool CheckNumbers()
    {
        return values.Count() > 0;
    }
}
```

### Static Local Functions

```csharp
public string ConcatenateNumbers(params int[] values)
{
    if (!CheckNumbers(values))
        throw new ArgumentException(nameof(values));

    return CombineNumbers(values);

    static string CombineNumbers(int[] values)
    {
        return $"[{String.Join(", ", values)}]";
    }

    static bool CheckNumbers(int[] values)
    {
        return values.Count() > 0;
    }
}
```

::: notes

The local function can be static since it doesn’t access any variables in the enclosing scope.

This example is functionally identical.

:::

### Static Local Function in Main

```csharp
static void Main(string[] args)
{
    Console.WriteLine(GetGreeting(args.FirstOrDefault()));

    static string GetGreeting(string name)
    {
        return $"Hello, {name ?? "Person"}!";
    }
}
```

### Property Initialization

* Very common to need default values for our properties
* Many times, we may have read-only properties that are only set in the constructor of our class

### Example Property Initialization

```csharp
public class User
{
    private string _domain = "github.com";

    string Domain {
        get { return _domain; }
    }
}
```

### Example Alternate Property Initialization

```csharp
public class User
{
    string Domain { get; }

    public User()
    {
        Domain = "github.com";
    }
}
```

### Auto-Property Initializers

* Declare an initial value for a property as part of the property declaration
  * Removes extra code necessary to initialize properties in the constructor

::: notes

<https://docs.microsoft.com/dotnet/csharp/whats-new/csharp-6#auto-property-initializers>

:::

### Example Auto-Property Initialization

```csharp
public class User
{
    string Domain { get; }  = "github.com";
}
```

### Expression-Bodied Members

* Use **lambda syntax** to define the body of a member
  * Methods
  * Properties
  * Constructors
  * Get/Set Accessors
  * Finalizers (Garbage Collection)

### Expression-Bodied Properties

```csharp
public class User
{
    public string FirstName { get; set; }

    public string Domain { get; set; }

    public string Email => $"{FirstName}@{Domain}";
}
```

### Expression-Bodied Methods

```csharp
public class User
{
    public string FirstName { get; set; }

    public string Domain { get; set; }

    public override string ToString() =>
        $"{Domain}//{FirstName}";
}
```

### Expression-Bodied Constructor

```csharp
public class User
{
    public string FirstName { get; set; }

    public string Domain = "SKILLMEUP.COM";

    public User(string firstName) =>
        FirstName = firstName;
}
```

### Expression-Bodied Accessors

```csharp
public class User
{
    private string _firstName;

    string FirstName
    {
        get => _firstName;
        set => _firstName = value ?? "STUDENT";
    }
}
```

### Expression-Bodied Main Entry Point

```csharp
static void Main(string[] args) =>
    Console.WriteLine("Hello, World!");
```

### Expression-Bodied ASP.NET Bootstrap

```csharp
public static async Task Main(string[] args) =>
    await Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        })
        .Build()
        .RunAsync();
```

### Tuples

* Types defined in a lightweight syntax
* Supported for a long time but the syntax was recently improved

![ ](media/tuples.png)

### Classic Tuple Syntax

```csharp
private Tuple<string, int> GetRoom()
{
    return new Tuple<string, int>("Breakout", 234);
}

var room = GetRoom();

Console.WriteLine($"Room: {room.Item1}");
Console.WriteLine($"Number: {room.Item2}");
```

### Improved Tuple Syntax

```csharp
private (string, int) GetRoom()
{
    return ("Breakout", 234);
}

(string name, int number) room = GetRoom();

Console.WriteLine($"Room: {room.name}");
Console.WriteLine($"Number: {room.number}");
```

### Tuple Name Inference

```csharp
string name = "Breakout";
int number = 234;

var room = (name, number);

Console.WriteLine($"Room: {room.name}");
Console.WriteLine($"Number: {room.number}");
```

### Tuple Explicit Naming

```csharp
(string first, string last) person = ("Sidney", "Andrews");

string fullname = $"{person.first} {person.last}";
```

### Tuple Explicit Naming (cont.)

```csharp
var person = (first: "Sidney", last: "Andrews");

string fullname = $"{person.first} {person.last}";
```

### Tuple decomposition

```csharp
private (string, int) GetRoom()
{
    return ("Breakout", 234);
}

(string name, int number) = GetRoom();

Console.WriteLine($"Room: {name}");
Console.WriteLine($"Number: {number}");
```

### Discards

* Dummy variables that are intentionally unused
  * Equivalent to variables that were never assigned
* Uses a special character as its name

![ ](media/discards.png)

### Using a Discard with a Tuple

```csharp
private (string, int) GetRoom()
{
    return ("Breakout", 234);
}

(_, int number) = GetRoom();

Console.WriteLine($"Number: {number}");
```

::: notes

Here, we don't need the room name so the code discards it immediately.

:::

## Demo: Building Complex Classes and Members

::: notes

TBD

:::

## Pattern Matching

* **Test** that a value has a certain schema (*or shape*)
* If the value "matches", then **extract** information
* All about improving the syntax for common checks

### Casting Values and the "is" Keyword

```csharp
if (shape is Rectangle)
{
    var r = shape as Rectangle;
    return r.HorizontalSide * r.VerticalSide;
}
else if (shape is Circle)
{
    var c = shape as Circle;
    return c.Radius * c.Radius * Math.PI;
}
else
{
    throw new ArgumentException("Shape not found", nameof(shape));
}
```

### Pattern Matching and "is" Keyword

```csharp
if (shape is Rectangle r)
    return r.HorizontalSide * r.VerticalSide;
else if (shape is Circle c)
    return c.Radius * c.Radius * Math.PI;
else
    throw new ArgumentException("Shape not found", nameof(shape));
```

### Pattern Matching and Switches

* Pattern matching changes functionality of a statement based on characteristics of the data
* Use matching with switch statements to build more concise “filters”

### Switch Syntax

```csharp
switch (shape)
{
    case Rectangle r:
        return r.HorizontalSide * r.VerticalSide;
    case Circle c:
        return c.Radius * c.Radius * Math.PI;
    default:
        throw new ArgumentException("Shape not found", nameof(shape));
}
```

### Pattern Matching Switch Syntax

```csharp
double area = shape switch
{
    Rectangle r => r.HorizontalSide * r.VerticalSide,
    Circle c => c.Radius * c.Radius * Math.PI,
    _ => throw new ArgumentException("Shape not found", nameof(shape))
};
```

### When Clauses

```csharp
double area = shape switch
{
    Rectangle r when r.HorizontalSide == 0 || r.VerticalSide == 0 => 0,
    Circle c when c.Radius == 0 => 0,
    Rectangle r => r.HorizontalSide * r.VerticalSide,
    Circle c => c.Radius * c.Radius * Math.PI,
    _ => throw new ArgumentException("Shape not found", nameof(shape))
};
```

### Example Enum

```csharp
public enum Location
{
    FloorOne = 0,
    FloorTwo,
    FloorThree
}
```

### Pattern Matches Switch Statement for Enum

```csharp
Location loc = Location.FloorTwo;

string location = loc switch
{
    Location.FloorOne => "Main Lobby",
    Location.FloorTwo => "Guest Rooms",
    Location.FloorThree => "Penthouse Suite",
    _ => "Not Found"
};

Console.WriteLine($"Location: {location}");
```

### Sample Class

```csharp
public class Student
{
    public string Name { get; set; }

    public double GPA { get; set; }
}
```

### Property Patterns

```csharp
Student student = new Student { Name = "Savannah" };

int grade = student switch
{
    { Name: "Jaiden" } => 90,
    { Name: "Jackson" } => 70,
    { Name: "Savannah" } => 85,
    _ => throw new ArgumentNullException(nameof(student))
};

Console.WriteLine($"{student.Name}'s Math Grade: {grade}");
```

### Tuple Patterns

```csharp
(string first, string last) = ("Jaiden", "Ashby");

string greeting = (first, last) switch
{
    ("Jaiden", _) => "Student of the year, Jaiden!",
    (_, "Andrews") => "Mr. Andrews",
    (_, _) => String.Empty
};

Console.WriteLine(greeting);
```

### Sample Types

```csharp
public enum Group
{
    Unknown = 0,
    UpperElementary,
    LowerElementary
}

public class Student
{
    public string FullName { get; set; }

    public int GradeLevel { get; set; }

    public void Deconstruct(out string name, out int grade)
    {
        name = FullName;
        grade = GradeLevel;
    }
}
```

### Sample Types (Improved)

```csharp
public enum Group
{
    Unknown = 0,
    UpperElementary,
    LowerElementary
}

public class Student
{
    public string FullName { get; set; }

    public int GradeLevel { get; set; }

    public void Deconstruct(out string name, out int grade) =>
        (name, grade) = (FullName, GradeLevel);
}
```

### Deconstruction

```csharp
var student = new Student { FullName = "Jaiden Ashby", GradeLevel = 5 };

Group group = student switch
{
    var (_, grade) when grade >= 3 => Group.UpperElementary,
    var (_, grade) when grade <= 2 => Group.LowerElementary,
    _ => Group.Unknown
};
```

## Demo: Implementing Pattern Matching

::: notes

TBD

:::
