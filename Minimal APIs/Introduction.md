| API                     | Description                 | Request body | Response body        |
| ----------------------- | --------------------------- | ------------ | -------------------- |
| GET /                   | Browser test, "Hello World" | None         | Hell World!          |
| GET /todoitems          | Get all to-do items         | None         | Array of to-do items |
| GET /todoitems/complete | Get completed to-do items   | None         | Array of to-do items |
| GET /todoitems/{id}     | Get an item by ID           | None         | To-do item           |
| POST /todoitems         | Add a new item              | To-do item   | To-do item           |
| PUT /todoitems/{id}     | Update an exisiting item    | To-do item   | None                 |
| DELETE /todoitems/{id}  | Delete an item              | None         | None                 |


```
dotnet new webapi -minimal -o TodoApi
cd TodoApi
code -r ../TodoApi
```

The following highlighted code creates a `WebApplicationBuilder` and a `WebApplication` with preconfigured defaults. 

```
var builder = WebApplication.CreatedBuilder(args); 
var app = builder.Builder()
```

The following code creates an HTTP GET endpoint `/` which returns `Hello World`: 
```
app.MapGet("/", () => "Hello World!");
```

#### Working with ports 

When a web app created, `Properties/launchSettings.json` file is created that specifics teh ports the app responds to. 

```
var app = WebApplication.Create(args);

app.MapGet("/", () => "Hello World!");

app.Run("http://localhost:3000");
```

In the preceding code, the app responds to port 3000.

#### Multiple ports 
```
var app = WebApplication.Create(args);

app.Urls.Add("http://localhost:3000");
app.Urls.Add("http://localhost:4000");

app.MapGet("/", () => "Hello World");

app.Run();
```

#### Set the port from the command line 
```
dotnet run --urls="https://localhost:7777"
```

#### Read the port from environment 
```
var app = WebApplication.Create(args);

var port = Environment.GetEnvironmentVariable("PORT") ?? "3000";

app.MapGet("/", () => "Hello World");

app.Run($"http://localhost:{port}");
```

The preferred way to set the port from the environment is to use the `ASPNETCORE_URLS` environment variable, which is hsown in the following section. 

#### Set the ports via teh ASPNETCORE_URLS environment variable 

The `ASPNETCORE_URLS` environment varibale is available to set the port 

```
ASPNETCORE_URLS=http://localhost:3000
```

`ASPNETCORE_URLS` supports multiple URLs: 

```
ASPNETCORE_URLS=http://localhost:3000;https://localhost:5000
```

Listen on all interfaces

The following samples demonstrate listening on all interfaces 

```
var app = WebApplication.Create(args);

app.Urls.Add("http://*:3000");
//app.Urls.Add("http://+:3000");
//app.Urls.Add("http://0.0.0.0:3000");

app.MapGet("/", () => "Hello World");

app.Run();
```

Listen on all interfaces using ASPNETCORE_URLS

the preceding samples can use `ASPNETCORE_URLS`

```
ASPNETCORE_URLS=http://*:3000;https://+:5000;http://0.0.0.0:5005
```

Specify HTTPS using a custom certificate 

the following sections show how ot specify the custom certificate using the `appsettings.json` file and via configuration. 

Specify the custom certificate with apptsettings.json 

```
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "Kestrel": {
    "Certificates": {
      "Default": {
        "Path": "cert.pem",
        "KeyPath": "key.pem"
      }
    }
  }
}
```

Specify the custom certificate via configuration 

```
var builder = WebApplication.CreateBuilder(args);

// Configure the cert and the key
builder.Configuration["Kestrel:Certificates:Default:Path"] = "cert.pem";
builder.Configuration["Kestrel:Certificates:Default:KeyPath"] = "key.pem";

var app = builder.Build();

app.Urls.Add("https://localhost:3000");

app.MapGet("/", () => "Hello World");

app.Run();
```

Using the certificate APIs

```
using System.Security.Cryptography.X509Certificates;

var builder = WebApplication.CreateBuilder(args);

builder.WebHost.ConfigureKestrel(options =>
{
    options.ConfigureHttpsDefaults(httpsOptions =>
    {
        var certPath = Path.Combine(builder.Environment.ContentRootPath, "cert.pem");
        var keyPath = Path.Combine(builder.Environment.ContentRootPath, "key.pem");

        httpsOptions.ServerCertificate = X509Certificate2.CreateFromPemFile(certPath, 
                                         keyPath);
    });
});

var app = builder.Build();

app.Urls.Add("https://localhost:3000");

app.MapGet("/", () => "Hello World");

app.Run();
```

Read the environment 

```
var app = WebApplication.Create(args);

if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/oops");
}

app.MapGet("/", () => "Hello World");
app.MapGet("/oops", () => "Oops! An error happened.");

app.Run();
```

For more information using the environment, see [Use multiple environments in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/environments?view=aspnetcore-6.0)

__Configuration 

The following code reads from the configuration system 

```
var app = WebApplication.Create(args);

var message = app.Configuration["HelloKey"] ?? "Hello";

app.MapGet("/", () => message);

app.Run();
```

For more information, see [Configuration in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-6.0)

__Logging 

The following code writes a message to the log on the application startup

```
var app = WebApplication.Create(args);

app.Logger.LogInformation("The app started");

app.MapGet("/", () => "Hello World");

app.Run();
```

For more information, see [Logging in .NET Core and ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/logging/?view=aspnetcore-6.0)

---
__Acces the Dependency Injection (DI) container 

The following code shows how to get services from DI container during application startup

```
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

For more information, see [Dependency injection in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-6.0).

---
#### WebApplicationBuilder 

This section contains sample code using [WebApplicationBuilder](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.webapplicationbuilder).

Change the content root, application name and environment 

```
var builder = WebApplication.CreateBuilder(new WebApplicationOptions
{
    ApplicationName = typeof(Program).Assembly.FullName,
    ContentRootPath = Directory.GetCurrentDirectory(),
    EnvironmentName = Environments.Staging,
    WebRootPath = "customwwwroot"
});

Console.WriteLine($"Application Name: {builder.Environment.ApplicationName}");
Console.WriteLine($"Environment Name: {builder.Environment.EnvironmentName}");
Console.WriteLine($"ContentRoot Path: {builder.Environment.ContentRootPath}");
Console.WriteLine($"WebRootPath: {builder.Environment.WebRootPath}");

var app = builder.Build();
```

[WebApplication.CreateBuilder](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.webapplication.createbuilder) initializes a new instance of the [WebApplicationBuilder](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.webapplicationbuilder) class with preconfigured defaults.

For more information, see [ASP.NET Core fundamentals overview](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/?view=aspnetcore-6.0)

---
__Change the content root, app name, and environment by environment variables or command line 

The following table shows the environment variable and command-line argument used to change the content root, app name, and environment:

| feature          | Environment variable       | Command-line argument |
| ---------------- | -------------------------- | --------------------- |
| Application name | ASPNETCORE_APPLICATIONNAME | --applicationName     |
| Environment name | ASPNETCORE_ENVIRONMENT     | --environment         |
| Content root     | ASPNETCORE_CONTENTROOT     | --contentRoot         |

---
__Add configuration providers 

```
var builder = WebApplication.CreateBuilder(args);

builder.Configuration.AddIniFile("appsettings.ini");

var app = builder.Build();
```

For detailed information, see [File configuration providers](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-6.0#file-configuration-provider) in [Configuration in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-6.0).

---
__Read configuraiton 

By default the `WebApplicationBuilder` reads configuration from multiple sources, including 
- `appSettings.json` and `appSettings.{environment}.json`
- Environment variables
- The command line 

For a complete list of configuration sources read, see [Default configuration](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-6.0#default-configuration) in [Configuration in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-6.0)

The following code reads `HelloKey` from configuration and displays the value at the `/` endpoint. If the configuration value is null, "Hello" is assigned to `message`:

```
var builder = WebApplication.CreateBuilder(args);

var message = builder.Configuration["HelloKey"] ?? "Hello";

var app = builder.Build();

app.MapGet("/", () => message);

app.Run();
```

---
__Read the environment 

```
var builder = WebApplication.CreateBuilder(args);

var message = builder.Configuration["HelloKey"] ?? "Hello";

var app = builder.Build();

app.MapGet("/", () => message);

app.Run();
```

---
__Add logging providers 

```
var builder = WebApplication.CreateBuilder(args);

// Configure JSON logging to the console.
builder.Logging.AddJsonConsole();

var app = builder.Build();

app.MapGet("/", () => "Hello JSON console!");

app.Run();
```

---
__Add services 

```
var builder = WebApplication.CreateBuilder(args);

// Add the memory cache services.
builder.Services.AddMemoryCache();

// Add a custom scoped service.
builder.Services.AddScoped<ITodoRepository, TodoRepository>();
var app = builder.Build();
```

---
__Customize the IHostBuilder 

Existing extension methods on [IHostBuilder](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.hosting.ihostbuilder) can be accessed using the [Host property](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.hosting.ihostbuilder.properties#microsoft-extensions-hosting-ihostbuilder-properties):

```
var builder = WebApplication.CreateBuilder(args);

// Wait 30 seconds for graceful shutdown.
builder.Host.ConfigureHostOptions(o => o.ShutdownTimeout = TimeSpan.FromSeconds(30));

var app = builder.Build();

app.MapGet("/", () => "Hello World!");

app.Run();
```

---
__Customize the IWebHostBuilder 

Extension methods on [IWebHostBuilder](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder) can be accessed using the [WebApplicationBuilder.WebHost](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.webapplicationbuilder.webhost#microsoft-aspnetcore-builder-webapplicationbuilder-webhost) property.

```
var builder = WebApplication.CreateBuilder(args);

// Change the HTTP server implemenation to be HTTP.sys based
builder.WebHost.UseHttpSys();

var app = builder.Build();

app.MapGet("/", () => "Hello HTTP.sys");

app.Run();
```

---
__Change the web root 

By default, the web root is relative to the content root in the `wwwroot` folder. Web root is where the static files middleware looks for static files. Web root can be changed with `WebHostOptions`, the command line, or with the [UseWebRoot](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.hosting.hostingabstractionswebhostbuilderextensions.usewebroot) method:

```
var builder = WebApplication.CreateBuilder(new WebApplicationOptions
{
    Args = args,
    // Look for static files in webroot
    WebRootPath = "webroot"
});

var app = builder.Build();

app.Run();
```

---
__Custom dependency injection (DI) container 

The following example uses [Autofac](https://autofac.readthedocs.io/en/latest/integration/aspnetcore.html):

```
var builder = WebApplication.CreateBuilder(args);

builder.Host.UseServiceProviderFactory(new AutofacServiceProviderFactory());

// Register services directly with Autofac here. Don't
// call builder.Populate(), that happens in AutofacServiceProviderFactory.
builder.Host.ConfigureContainer<ContainerBuilder>(builder => builder.RegisterModule(new MyApplicationModule()));

var app = builder.Build();
```

---
__Add Middleware 

Any existing ASP.NET Core middleware can be configured on the `WebApplication`:

```
var app = WebApplication.Create(args);

// Setup the file server to serve static files.
app.UseFileServer();

app.MapGet("/", () => "Hello World!");

app.Run();
```

---
__Developer exception page 

[WebApplication.CreateBuilder](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.webapplication.createbuilder) initializes a new instance of the [WebApplicationBuilder](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.webapplicationbuilder) class with preconfigured defaults. The developer exception page is enabled in the preconfigured defaults. When the following code is run in the [development environment](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/environments?view=aspnetcore-6.0), navigating to `/` renders a friendly page that shows the exception.

```
var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

app.MapGet("/", () =>
{
    throw new InvalidOperationException("Oops, the '/' route has thrown an exception.");
});

app.Run();
```

---
#### ASP.NET Core Middleware 

![[Pasted image 20220730235233.png]]

---
__Request handling 

The following sections cover routing, parameter binding, and response. 

---
__Routing 

A configured `WebApplication` supports `Map{Verb}` and [MapMethods](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.endpointroutebuilderextensions.mapmethods):

``` CSHARP
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

___
__Route Handlers 

Route handlers are methods that execute when the route matches. Route handlers can be a function or any shape, including synchronous or asynchronous. Route handlers can be a lambda expression, a local function, an instance method or a static method.

___
__Lambda expression 

```CSHARP
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/inline", () => "This is an inline lambda");

var handler = () => "This is a lambda variable";

app.MapGet("/", handler);

app.Run();
```

---
__Local function 

```CSHARP
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

string LocalFunction() => "This is local function";

app.MapGet("/", LocalFunction);

app.Run();
```

---
__Instance method 

```CSHARP
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

___
__Static method 

```CSHARP
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

---
__Name routes and link generation 

```CSHARP
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/hello", () => "Hello named route")
   .WithName("hi");

app.MapGet("/", (LinkGenerator linker) => 
        $"The link to the hello route is {linker.GetPathByName("hi", values: null)}");

app.Run();
```

The preceding code displays `The link to the hello route is /hello` from the `/` endpoint.

---
__Route Parameters 

Route parameters can be captured as part of the route pattern definition:

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/users/{userId}/books/{bookId}", 
    (int userId, int bookId) => $"The user id is {userId} and book id is {bookId}");

app.Run();
```

The preceding code returns `The user id is 3 and book id is 7` from the URI `/users/3/books/7`.

The route handler can declare the parameters to capture. When a request is made a route with parameters declared to capture, the parameters are parsed and passed to the handler. This makes it easy to capture the values in a type safe way. In the preceding code, `userId` and `bookId` are both `int`.

In the preceding code, if either route value cannot be converted to an `int`, an exception is thrown. The GET request `/users/hello/books/3` throws the following exception:

**`BadHttpRequestException: Failed to bind parameter "int userId" from "hello"`**

---
__Wildcard and catch all routes 

The following catch all route returns `Routing to hello` from the `/posts/hello' endpoint:

```c#
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/posts/{*rest}", (string rest) => $"Routing to {rest}");

app.Run();
```
---
__Route constraints 

```c#
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/todos/{id:int}", (int id) => db.Todos.Find(id));
app.MapGet("/todos/{text}", (string text) => db.Todos.Where(t => t.Text.Contains(text)));
app.MapGet("/posts/{slug:regex(^[a-z0-9_-]+$)}", (string slug) => $"Post {slug}");

app.Run();
```
| Route Template | Example Matching URI | 
| -------------- | -------------------- |
|`/todos/{id:int}`|`/todos/1`|
|`/todos/{text}`|`/todos/something`|
|`/posts/{slug:regex(^[a-z0-9_-]+$)}`|`/posts/mypost`|

