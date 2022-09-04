### Overview 
| API                     | Description                 | Requested body | Response body        |
| ----------------------- | --------------------------- | -------------- | -------------------- |
| GET /                   | Browser test, "Hello World" | None           | Hello World!         |
| GET /todoitems          | Get all to-do items         | None           | Array of to-do items |
| Get /todoitems/complete | Get completed to-do items   | None           | Array of to-do items |
| Get /todoitems/{id}     | Get an item by ID           | None           | To-do item           |
| POST /todoitems         | Add a new item              | To-do item     | To-do item           |
| PUT /todoitems/{id}     | Update an existing item     | To-do item     | None                 |
| DELETE /todoitems/{id}  | Delete an item              | None           | None                 |
|                         |                             |                |                      |

### Create a Web API project 
```.NET CLI
dotnet new webapi -minimal -o TodoApi
cd TodoApi
code -r ../TodoApi
```



[Tutorial: Create a minimal web API with ASP.NET Core | Microsoft Docs](https://docs.microsoft.com/en-us/aspnet/core/tutorials/min-web-api?view=aspnetcore-6.0&tabs=visual-studio-code)

### Examine the code 

The `Program.cs` file contains the following code: 

```C#
var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
// Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

var summaries = new[]
{
    "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
};

app.MapGet("/weatherforecast", () =>
{
    var forecast = Enumerable.Range(1, 5).Select(index =>
       new WeatherForecast
       (
           DateTime.Now.AddDays(index),
           Random.Shared.Next(-20, 55),
           summaries[Random.Shared.Next(summaries.Length)]
       ))
        .ToArray();
    return forecast;
})
.WithName("GetWeatherForecast");

app.Run();

internal record WeatherForecast(DateTime Date, int TemperatureC, string? Summary)
{
    public int TemperatureF => 32 + (int)(TemperatureC / 0.5556);
}
```


#### Working with ports 
```csharp
var app = WebApplication.Create(args);

app.MapGet("/", () => "Hello World!");

app.Run("http://localhost:3000");
```

#### Read the port from environment
---

#### Logging
---
```csharp
var app = WebApplication.Create(args);

app.Logger.LogInformation("The app started");

app.MapGet("/", () => "Hello World");

app.Run();
```


#### Access the Dependency Injection (DI) container
---
```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddScoped<SampleService>();

var app = builder.Build();

app.MapControllers();

using (var scope = app.Services.CreateScope())
{
    var sampleService = scope.ServiceProvider.GetRequiredService<SampleService>();
    sampleService.DoSomething();
}

app.Run();
```

## Request handling
---
#### Routing
---
```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", () => "This is a GET");
app.MapPost("/", () => "This is a POST");
app.MapPut("/", () => "This is a PUT");
app.MapDelete("/", () => "This is a DELETE");

app.MapMethods("/options-or-head", new[] { "OPTIONS", "HEAD" }, 
                          () => "This is an options or head request ");

app.Run();
```

#### Route Handlers 
---
Route hanlders are methods that execute when the route matches. Route handlers can be a function or any shape, including synchronous or asynchronous. Route handlers can be a lambda expression, a local function, an instance method or a static method 

- Lambda expression 
```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/inline", () => "This is an inline lambda");

var handler = () => "This is a lambda variable";

app.MapGet("/", handler);

app.Run();
```

- Local function 
```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

string LocalFunction() => "This is local function";

app.MapGet("/", LocalFunction);

app.Run();
```

- Instance method 
```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

var handler = new HelloHandler();

app.MapGet("/", handler.Hello);

app.Run();

class HelloHandler
{
    public string Hello()
    {
        return "Hello Instance method";
    }
}
```

- Static method 
```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", HelloHandler.Hello);

app.Run();

class HelloHandler
{
    public static string Hello()
    {
        return "Hello static method";
    }
}
```

#### Route Parameters
---
Route parameters can be captured as part of the route pattern definition:
```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/users/{userId}/books/{bookId}", 
    (int userId, int bookId) => $"The user id is {userId} and book id is {bookId}");

app.Run();
```

#### Route Constraints 
---
```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/todos/{id:int}", (int id) => db.Todos.Find(id));
app.MapGet("/todos/{text}", (string text) => db.Todos.Where(t => t.Text.Contains(text)));
app.MapGet("/posts/{slug:regex(^[a-z0-9_-]+$)}", (string slug) => $"Post {slug}");

app.Run();
```

#### Parameter Binding 
---
Parameter binding is the process of converting request data into strongly typed parameters that are expressed by route handlers. A binding source determines where parameters are bound from. Binding sources can be explicit or inferred based on HTTP method and parameter type.

Supported binding sources:

-   Route values
-   Query string
-   Header
-   Body (as JSON)
-   Services provided by dependency injection
-   Custom

#### Explicit Parameter Binding
---
```csharp 
using Microsoft.AspNetCore.Mvc;

var builder = WebApplication.CreateBuilder(args);

// Added as service
builder.Services.AddSingleton<Service>();

var app = builder.Build();


app.MapGet("/{id}", ([FromRoute] int id,
                     [FromQuery(Name = "p")] int page,
                     [FromServices] Service service,
                     [FromHeader(Name = "Content-Type")] string contentType) 
                     => {});

class Service { }

record Person(string Name, int Age);
```

