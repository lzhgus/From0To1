### [Home · jbogard/MediatR Wiki (github.com)](https://github.com/jbogard/MediatR/wiki)

```C# 
new Container(cfg => cfg.Scan(scanner => {
    scanner.TheCallingAssembly();
    scanner.AddAllTypesOf(typeof(IRequestHandler<,>));
    scanner.AddAllTypesOf(typeof(INotificationHandler<>));
});
```

---
#### Basics 
MediatR has two kinds of messages it dispatches: 
- Request/Response messages, dispatched to a single handler 
- Notification messages, dispatched to multiple handlers 

##### Request/response 
The request/response interface handles both command and query scenarios. First creat a message: 
```c#
public class Ping: IRequest<String> {}
```

Next, create a hander: 

```c#
public class PingHandler : IRequestHandler<Ping, string> 
{
	public Task<string> Handle(Ping request, CancellationToken cancellationToken)
	{
		return Task.FromResult("Pong")
	}
}
```

Finnally, send a message throught the mediator: 

```C#
Var response = await mediator.Send(new Ping()): 
Debug.WriteLine(response); // "Pong"

```

> Feature.cs AppMessageConsumer.cs await _mediator.Send(x)


In the case your message does not requrrie a response, user the AsyncRequestHandlerTRequest base class: 

```c#
public class OneWay: IRequest()
public class OneWayHandlerWithBaseClass: AsyncRequestHanlder<OneWay>
{
	protected override string Handle(OneWay request, CancellationToken cancellationToken)
	{
		// Twiiddle thumbs	
	}
}
```

Or if the request is completely synchronus, inherit from the base ReqeustHandler Class

```c#
public class override string Handle(Ping request)
{
	return "Pong"; 
}
```

##### Request types 

There are two flavors of requests in MediatR - ones that return a value, and ones that do not 

- `IRequest<T>` - the requrest returns a value 
- `IRequest` - the request does not return a value 

To simplify the execution pipeline, `IRequest` inherits `IRequest<Unit>` whwere `Unit` represents a terminal/ignored return type. 

Each request type has its own handler interface, as well as some helper base classes: 

- `IRequestHandler<T, U>` - implement this and return `Task<U>`
- `RequestHandler<T, U` - inherit this and return U 

Then for requests without return values: 

- `IRequestHandler<T>` - implement this and you will return `Task<Unit`
- `AsyncRequestHandler<T>` - intherit this and your will return `Task`
- `RequestHanlder<T>` - inherit this and you will return nothing (`void`)

---
#### Streams and AsyncEnumerables 

To create a stream from a reqeust, first implement the stream request and its response 

- `IStreamRequest<TResponse>`

Stream request handlers are separate from the normal `IRequestHandler` and require implementing: 

- `IStreamRequestHandler<TReqeust, TResponse>`

Unlike normal request handlers that return a single `TResponse`, a stream handler returns an `IAsyncEnumerable<TResponse`: 

```c#
IAsyncEnumerable<TResponse> Handle(TRequest requrest, CancellationToken cancellationToken); 
```

To creat a stream request handler, creat a class that implements `IStreamReqeustHandler<TRequest, TResponse` and implement the above handle method.

---
#### Notifications

For notifications, first creat your notifiaction messages: 

```c#
public class Ping: INotification {}
```

Next, create zero or more handlers for your notification: 

```c#
public class Pong1: INotificationHandler<Ping>
{
	public Task Handle(Ping notification, CancellationToken cancellationToken)
	{
		Debug.WriteLine("Pong 1"); 
		return Task.CompletedTask;
	}
}

public class Pong2: INotificationHandler<Ping> 
{
	public Task Handle(Ping notification, CancellationToken cancellationToken)
	{
		Debug.WriteLine("Pong 2"); 
		return Task.CompletedTask;
	}
}
```

Finally, publish your message via the mediator: 

```C#
await mediator.Publish(new Ping()); 
```

##### Publish strategies

The default implementation of PUblish loops throught the notification handlers and awaits weach one. This ensures each handler is run after one another. 

Depending on your use-case for publishing notifications, you might need a different strategy for handling the notifications. Maybe you want to publish all notifications in parallel, or wrap each notification handler with your ownb exception handling logic. 

A few example implementations can be found in [MediatR/samples/MediatR.Examples.PublishStrategies at master · jbogard/MediatR (github.com)](https://github.com/jbogard/MediatR/tree/master/samples/MediatR.Examples.PublishStrategies). This shows four different strategies documented on the [PublishStrategy]([MediatR/PublishStrategy.cs at master · jbogard/MediatR (github.com)](https://github.com/jbogard/MediatR/blob/master/samples/MediatR.Examples.PublishStrategies/PublishStrategy.cs))enum. 

---
#### Polymorphic dispatch 

Handler interfaces are contravariant: 

> [Covariance and Contravariance (C#) | Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/covariance-contravariance/)

```c#
public interface IReqeustHandler<in TReqeust, TResponse> where TRequest: IRequest<TRequest>
{
	Task<TResponse> Handle(TReqeust messge, CancellationToken cancellationToken); 
}

public interface INotificationHandler<in TNotification>
{
	Task Handler(TNotification notification, CancellationTOken cancellationToken); 
}
```

Containers that support generic variance will dispatch accordingly. For example, you can have an `INotifiactionhandler<INotification>` to handle all notifications. 

---
#### Async 

Send/publish are asynchronous from the `IMediator` side, with corresponding synchronous and asynchronous-based interfaces/base classes for reqeusts/notification handlers. 

Your handlers can use the async/await keywords as long as the work is awaitable: 

```c#
public class PingHandler : IReqeustHandler<Ping, Pong> 
{
	public asyc Task<Pong> Handle(Ping request, CancellationToken cancellationTOken)
	{
	await DoPong()}; // whatever DoPong dose 
}
```

you will also need to register these handlers with the IoC container of your choice, similar to the synchronous handlers whown above. 


#### Exceptions handling 
---
##### Exception handler pipeline step 

Eaxception handler implemented by using IPipelineBehavior concept. It requires to add the `RequestEx ceptionProcessorBehavior` to the request execution Pipeline. 

Handlers types 

There are two flavors of exception handlers in teh MediatR - ones that specify exact excetption, and ones that do not: 

- `IReqeustExceptionHanlder<in TReqeust, TResponse, IException` - implement to handle exceptions that inherits from `TException` and were thrown from any requsts that inherit `TRequest`; 
- `IReqeustExceptionHandler<in TReqeust, TResponse>` - implement to handle all exceptions which were thrown from any requests that inherit `TRequest`. 

To simplify and generalize the exceptions handling, `IReqeustExceptionHandler<in TReqeust, Tresponse>` inherits `IReqeustEDExceptionHandler<Treqeust, TResponse, Exception>`

Sereval abstractions exists in the MediatR to simplify exception handler creation: 

- `AsyncReqeustExceptionHandler<TRequest, TResponse>` - inherit this to asynchronously handle any exception which were thrown from any requsts that inherits `TRequest`;
- `ReqeustExceptionHandler<TRequest, TResponse, TException>` - inherit this to synchronously handle any exception that inherits `TException` and were thrown from any reqeusts that inherits `TRequest`; 
- `RequestExceptionHandler<TReqeust, TResponse>` - inherit this to synchronously handle any exception which were thrown from any requests that inherits `TReqeust`; 

##### Exception action pipeline step 

todo 


#### Handlers and actions priority execution

All available handlers/actions will be sorted by applying next rules:

-   The handler/action has a higher priority if it belongs to the current assembly (same assembly with request) and the other is not. If none of the objects belong to the current assembly, they can be considered equal. If both objects belong to the current assembly, they can't be compared only by this criterion - compare by next rule;
-   The handler/action has a higher priority if it belongs to the current/child request namespace and the other is not. If both objects belong to the current/child request namespace, they can be considered equal. If none of the objects belong to the current/child request namespace, they can't be compared by this criterion - compare by next rule;
-   The handler/action has a higher priority if it namespace is part of the current location (request namespace) and the other is not. If both objects are part of the current location, the closest has higher priority. If none of the objects are part of the current location, they can be considered equal.