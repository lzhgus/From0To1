

[Best 6 Dependency Injection Containers (IOC) Comparison (codingsight.com)](https://codingsight.com/configuation-comparison-dependency-injection-containers/)

[Use dependency injection in .NET | Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/core/extensions/dependency-injection-usage)

[Dependency injection in ASP.NET Core | Microsoft Docs](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-6.0)

# Constructor Injection 

#### Autofac Constructor Injection 

```c#
var builder = new ContainerBuilder(); 
builder.RegisterType<AuthorRepositoryCtro>().As<IAuthorRepository>(); builder.RegisterType<BookRepositoryCtro>().As<IBookRepository>(); builder.RegisterType<ConsoleLog>().As<ILog>(); 
var container = builder.Build();
```

#### StructureMap Constructor Injection 

```c#
var container = new Container(); 
container.Configure(c => { 
c.For<IAuthorRepository>().Use<AuthorRepositoryCtro>(); 
c.For<IBookRepository>().Use<BookRepositoryCtro>(); 
c.For<ILog>().Use<ConsoleLog>(); });
```

#### Simple Injector Constructor Injection 

```c#
var container = new Container(); 
container.Register<IAuthorRepository, AuthorRepositoryCtro>(); container.Register<IBookRepository, BookRepositoryCtro>(); 
container.Register<ILog, ConsoleLog>();
```



