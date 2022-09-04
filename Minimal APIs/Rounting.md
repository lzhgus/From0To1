##### Paramter Binding 
---
Paramter binding is the process of converting request data into strongly typed parameters that are expressed by route handlers. A binding source determines where parameters are bound from. Binding sources can be explicit or inferred based on HTTP method and paramter type.

Supported binding sources:

-   Route values
-   Query string
-   Header
-   Body (as JSON)
-   Services provided by dependency injection
-   Custom

```c#
var builder = WebApplication.CreateBuilder(args);

// Added as service
builder.Services.AddSingleton<Service>();

var app = builder.Build();

app.MapGet("/{id}", (int id,
                     int page,
                     [FromHeader(Name = "X-CUSTOM-HEADER")] string customHeader,
                     Service service) => { });

class Service { }
```

The following table shows the relationship between the parameters used in the preceding example and the associated binding sources. 

| Paramter       | Binding Source                   |
| -------------- | -------------------------------- |
| `id`           | route value                      |
| `page`         | query string                     |
| `customHeader` | header                           |
| `service`      | Provided by dependency injection |

The HTTP methods `GET`, `HEAD`, `OPTIONS`, and `DELETE` don't implicitly bind from body. To bind from body (as JSON) for these HTTP methods, [bind explicitly](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-6.0#explicit-parameter-binding) with [`[FromBody]`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.frombodyattribute) or read from the [HttpRequest](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.http.httprequest).

The following example POST route handler uses a binding source of body (as JSON) for the `person` parameter:

```c#
var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

app.MapPost("/", (Person person) => { });

record Person(string Name, int Age);
```

The paramters in the preceding examples are all bound from request data automaticallly. To demonstrate the convenience that parameters binding provides, the following example route handlers show how to read request data directly from the request. 

```c#
app.MapGet("/{id}", (HttpRequest request) =>
{
    var id = request.RouteValues["id"];
    var page = request.Query["page"];
    var customHeader = request.Headers["X-CUSTOM-HEADER"];

    // ...
});

app.MapPost("/", async (HttpRequest request) =>
{
    var person = await request.ReadFromJsonAsync<Person>();

    // ...
});
```
---
##### Explicit Paramter Binding 
```c#
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
| Paramter    | Binding Source                      |
| ----------- | ----------------------------------- |
| id          | route value with the same id        |
| page        | query string with the name 'p'      |
| service     | Provided by dependency injection    |
| contentType | header with the name 'Content-Type' |

---
##### Paramter binding with DI 
Parameter binding for minimal APIs binds parameters through [dependency injection](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-6.0) when the type is configured as a service. It's not necessary to explicitly apply the [`[FromServices]`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.fromservicesattribute) attribute to a parameter. In the following code, both actions return the time:
```c#
using Microsoft.AspNetCore.Mvc;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddSingleton<IDateTime, SystemDateTime>();

var app = builder.Build();

app.MapGet("/",   (               IDateTime dateTime) => dateTime.Now);
app.MapGet("/fs", ([FromServices] IDateTime dateTime) => dateTime.Now);
app.Run();
```

---
##### Optional paramters 
Parameters declared in route handlers are treated as required:

-   If a request matches the route, the route handler only runs if all required parameters are provided in the request.
-   Failure to provide all required parameters results in an error.

```c#
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/products", (int pageNumber) => $"Requesting page {pageNumber}");

app.Run();
```

| URI                      | result                                                                                             |
| ------------------------ | -------------------------------------------------------------------------------------------------- |
| `/products?pageNumber=3` | 3 returned                                                                                         |
| `/products`              | `BadHttpRequestException`: Required parameter "int pageNumber" was not provided from query string. |
| `/products/1`            | HTTP 404 error, no matching route                                                                  |

To name `pageNumber` optional, define the type as optional or provide a default value: 

```c#
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/products", (int? pageNumber) => $"Requesting page {pageNumber ?? 1}");

string ListProducts(int pageNumber = 1) => $"Requesting page {pageNumber}";

app.MapGet("/products2", ListProducts);

app.Run();
```

| URI                      | result     |
| ------------------------ | ---------- |
| `/products?pageNumber=3` | 3 returned |
| `/products`              | 1 returned |
| `/products2`             | 1 returned |

The preceding nullable and default value applies to all sources: 
```c#
var builder = WebApplication.CreateBuilder(args); 
var app = builder.Build();

app.MapPost("/products", (Product? product) => { }); 

app.Run();
```

The preceding code calls the method with a null product if no request body is sent.

**NOTE**: If invalid data is provided and the parameter is nullable, the route handler is _**not**_ run.

```c#
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/products", (int? pageNumber) => $"Requesting page {pageNumber ?? 1}");

app.Run();
```

| URI                      | result                                                                                 |
| ------------------------ | -------------------------------------------------------------------------------------- |
| `/products?pageNumber=3` | 3 returned                                                                             |
| `/products`              | 1 returned                                                                             |
| `/products/two`          | `BadHttpRequestException`: Failed to bind paramter `Nullable<int> pageNumber` from two |

---
#### Special types 

The following types are bound without explicit attributes: 

[HttpContext](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.http.httpcontext): The context which holds all the information about the current HTTP request or response:

```c#
app.MapGet("/", (HttpContext context) => context.Response.WriteAsync("Hello World"));
```

[HttpRequest](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.http.httprequest) and [HttpResponse](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.http.httpresponse): The HTTP request and HTTP response:

```c#
app.MapGet("/", (HttpRequest request, HttpResponse response) =>
    response.WriteAsync($"Hello World {request.Query["name"]}"));
```

[CancellationToken](https://docs.microsoft.com/en-us/dotnet/api/system.threading.cancellationtoken): The cancellation token associated with the current HTTP request:

```c#
app.MapGet("/", async (CancellationToken cancellationToken) => 
    await MakeLongRunningRequestAsync(cancellationToken));
```

[ClaimsPrincipal](https://docs.microsoft.com/en-us/dotnet/api/system.security.claims.claimsprincipal): The user associated with the request, bound from [HttpContext.User](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.http.httpcontext.user):

```c#
app.MapGet("/", (ClaimsPrincipal user) => user.Identity.Name);
```

---
#### Custom Binding 

There are two ways to customize paramter binding: 

1. For route, query, and header binding sources, bind custom types by adding a static `TryParse` method for the type. 
2. Control the binding process by implementing a `BindingAsync` method on a type. 

__TryParse 

`TryParse` has two APIs: 

```c#
public static bool TryParse(string value, out T result);
public static bool TryParse(string value, IFormatProvider provider, out T result);
```

The following code displays `Point: 12.3, 10.1` with the URI `/map?Point=12.3,10.1`

```c#
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// GET /map?Point=12.3,10.1
app.MapGet("/map", (Point point) => $"Point: {point.X}, {point.Y}");

app.Run();

public class Point
{
    public double X { get; set; }
    public double Y { get; set; }

    public static bool TryParse(string? value, IFormatProvider? provider,
                                out Point? point)
    {
        // Format is "(12.3,10.1)"
        var trimmedValue = value?.TrimStart('(').TrimEnd(')');
        var segments = trimmedValue?.Split(',',
                StringSplitOptions.RemoveEmptyEntries | StringSplitOptions.TrimEntries);
        if (segments?.Length == 2
            && double.TryParse(segments[0], out var x)
            && double.TryParse(segments[1], out var y))
        {
            point = new Point { X = x, Y = y };
            return true;
        }

        point = null;
        return false;
    }
}
```

__BindAsync 

`BindAsync` has the following APIs: 

```c#
public static ValueTask<T?> BindAsync(HttpContext context, ParameterInfo parameter);
public static ValueTask<T?> BindAsync(HttpContext context);
```

The following code displays `SortBy:xyz, SortDirection:Desc, CurrentPage:99` with the URI `/products?SortBy=xyz&SortDir=Desc&Page=99`:

```c#
using System.Reflection;

var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// GET /products?SortBy=xyz&SortDir=Desc&Page=99
app.MapGet("/products", (PagingData pageData) => $"SortBy:{pageData.SortBy}, " +
       $"SortDirection:{pageData.SortDirection}, CurrentPage:{pageData.CurrentPage}");

app.Run();

public class PagingData
{
    public string? SortBy { get; init; }
    public SortDirection SortDirection { get; init; }
    public int CurrentPage { get; init; } = 1;

    public static ValueTask<PagingData?> BindAsync(HttpContext context,
                                                   ParameterInfo parameter)
    {
        const string sortByKey = "sortBy";
        const string sortDirectionKey = "sortDir";
        const string currentPageKey = "page";

        Enum.TryParse<SortDirection>(context.Request.Query[sortDirectionKey],
                                     ignoreCase: true, out var sortDirection);
        int.TryParse(context.Request.Query[currentPageKey], out var page);
        page = page == 0 ? 1 : page;

        var result = new PagingData
        {
            SortBy = context.Request.Query[sortByKey],
            SortDirection = sortDirection,
            CurrentPage = page
        };

        return ValueTask.FromResult<PagingData?>(result);
    }
}

public enum SortDirection
{
    Default,
    Asc,
    Desc
}
```
---

##### Binding failures 
When binding fails, the framework logs a debug message and returns various status codes to the client depending on the failure mode. 

| Failure mode                              | Nullable Parameter Type | Binding Source     | Status code |
| ----------------------------------------- | ----------------------- | ------------------ | ----------- |
| {ParameterType}.TryParse returns false    | yes                     | route/query/header | 400         |
| {ParameterType}.BindAsync returns null    | yes                     | Custom             | 400         |
| {ParameterType}.BindAsync throws          | does not matter         | custom             | 500         |
| Failure to deserialize JSON body          | does not matter         | body|  400           |
| Wrong content type (not application/json) | does not matter         | body               | 415            |

---
##### Binding Precedence
The rules for determining a binding source from a parameter: 
1.  Explicit attribute defined on parameter (From* attributes) in the following order:
    1.  Route values: [FromRoute](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.fromrouteattribute)
    2.  Query string: [FromQuery](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.fromqueryattribute)
    3.  Header: [FromHeader](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.fromheaderattribute)
    4.  Body: [FromBody](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.frombodyattribute)
    5.  Service: [FromServices](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.fromservicesattribute)
2.  Special types
    1.  `HttpContext`
    2.  `HttpRequest`
    3.  `HttpResponse`
    4.  `ClaimsPrincipal`
    5.  `CancellationToken`
3.  Parameter type has a valid `BindAsync` method.
4.  Parameter type is a string or has a valid `TryParse` method.
    1.  If the parameter name exists in the route template e.g. `app.Map("/todo/{id}", (int id) => {});`, then it's bound from the route.
    2.  Bound from the query string.
5.  If the parameter type is a service provided by dependency injection, it uses that service as the source.
6.  The parameter is from the body.

---
##### Customize JSON binding 
The body binding source uses [System.Text.Json](https://docs.microsoft.com/en-us/dotnet/api/system.text.json) for de-serialization. It is _**not**_ possible to change this default, but the binding can be customized using other techniques described previously. To customize JSON serializer options, use code similar to the following:

```C#
using Microsoft.AspNetCore.Http.Json;

var builder = WebApplication.CreateBuilder(args);

// Configure JSON options.
builder.Services.Configure<JsonOptions>(options =>
{
    options.SerializerOptions.IncludeFields = true;
});

var app = builder.Build();

app.MapPost("/products", (Product product) => product);

app.Run();

class Product
{
    // These are public fields, not properties.
    public int Id;
    public string? Name;
}
```

---
#### Response 
1. `IResult` based - This includes `Task<IResult` and `ValueTask<IResult>`
	1. The framework calls [IResult.ExecuteAsync](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.http.iresult.executeasync)
2. `string` - This includes `Task<string>` and `ValueTask<string>`
3. `T` (Any other type) - This includes `Task<T>` and `ValueTask<T>`

IResult return values 
```c#
app.MapGet("/hello", () => Results.Ok(new { Message = "Hello World" }));
```

The following example uses the built-in result types to customize the response: 
```c#
app.MapGet("/api/todoitems/{id}", async (int id, TodoDb db) =>
         await db.Todos.FindAsync(id) 
         is Todo todo
         ? Results.Ok(todo) 
         : Results.NotFound())
   .Produces<Todo>(StatusCodes.Status200OK)
   .Produces(StatusCodes.Status404NotFound);
```

Built-in result 

![[Pasted image 20220731225403.png]]

---
## Authorization 
Routes can be protected using authorization policies. These can be declared via the [Authorize](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authorization.authorizeattribute) attribute or by using the [RequireAuthorization](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.authorizationendpointconventionbuilderextensions.requireauthorization) method:

```c#
var builder = WebApplication.CreateBuilder(args); 
builder.Services.AddAuthorization(o => o.AddPolicy("AdminOnly", b => b.RequireClaim("admin", "true"))); 

app.MapGet("/auth", [Authorize] () => "This endpoint requries authorization");
```

The preceding code can be written with [RequireAuthorization](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.authorizationendpointconventionbuilderextensions.requireauthorization):
```c#
app.MapGet("/auth", () => "This endpoint requires authorization")
   .RequireAuthorization();
```

The following sample uses [policy-based authorization](https://docs.microsoft.com/en-us/aspnet/core/security/authorization/policies?view=aspnetcore-6.0):

```c#
app.MapGet("/admin", [Authorize("AdminsOnly")] () => "The /admin endpoint is for admins only.")
```

##### Allow unauthenticated users to access an endpoint 

The [AllowAnonymous](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.authorization.allowanonymousattribute) allows unauthenticated users to access endpoints:

```c#
app.MapGet("/login", [AllowAnonymous] () => "This endpoint is for all roles.");


app.MapGet("/login2", () => "This endpoint also for all roles.")
   .AllowAnonymous();
```

---
## CORS
Routes can be [CORS](https://docs.microsoft.com/en-us/aspnet/core/security/cors?view=aspnetcore-6.0) enabled using [CORS policies](https://docs.microsoft.com/en-us/aspnet/core/security/cors?view=aspnetcore-6.0#cors-policy-options). CORS can be declared via the [`[EnableCors]`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.cors.enablecorsattribute) attribute or by using the [RequireCors](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.corsendpointconventionbuilderextensions.requirecors) method. The following samples enable CORS:

```c#
const string MyAllowSpecificOrigins = "_myAllowSpecificOrigins";

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddCors(options =>
{
    options.AddPolicy(name: MyAllowSpecificOrigins,
                      builder =>
                      {
                          builder.WithOrigins("http://example.com",
                                              "http://www.contoso.com");
                      });
});

var app = builder.Build();
app.UseCors();

app.MapGet("/",() => "Hello CORS!");

app.Run();
```

---
## OpenAPI 
An app can describe the [OpenAPI specification](https://swagger.io/specification/) for route handlers using [Swashbuckle](https://www.nuget.org/packages/Swashbuckle.AspNetCore/).

The following code is a typical ASP.NET Core app with OpenAPI support:

```c#
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new() { Title = builder.Environment.ApplicationName,
                               Version = "v1" });
});

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI(c => c.SwaggerEndpoint("/swagger/v1/swagger.json",
                                    $"{builder.Environment.ApplicationName} v1"));
}

app.MapGet("/swag", () => "Hello Swagger!");

app.Run();
```
