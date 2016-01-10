# Using Dependency Injection (DI) to register and resolve application services

## Creating a service to assign request IDs
1. Create a folder in the application called `Services`
1. Create a new class file in the `Services` folder `RequestId`
1. In the file, create an interface `IRequestIdFactory` with a single method `string MakeRequestId()`
1. In the same file, create a class `RequestIdFactory` that implements `IRequestIdFactory` by using `Interlock.Increment` to make an increasing request ID
1. The file should look something like this:

  ``` C#
  public interface IRequestIdFactory
  {
      string MakeRequestId();
  }

  public class RequestIdFactory : IRequestIdFactory
  {
      private int _requestId;

      public string MakeRequestId() => Interlocked.Increment(ref _requestId).ToString();
  }
  ```

1. In the same file, create an interface `IRequestId` with a single property `string Id { get; }`
1. In the same file, create a class `RequestId` that implements `IRequestId` by taking an `IRequestIdFactory` in the constructor and calling its `MakeRequestId` method to get a new ID.
1. The whole file should now look something like this:

  ``` C#
  public interface IRequestIdFactory
  {
      string MakeRequestId();
  }
  
  public class RequestIdFactory : IRequestIdFactory
  {
      private int _requestId;
  
      public string MakeRequestId() => Interlocked.Increment(ref _requestId).ToString();
  }
  
  public interface IRequestId
  {
      string Id { get; }
  }
  
  public class RequestId : IRequestId
  {
      private readonly string _requestId;
  
      public RequestId(IRequestIdFactory requestIdFactory)
      {
          _requestId = requestIdFactory.MakeRequestId();
      }
  
      public string Id => _requestId;
  }
  ```

## Register the request ID service in DI
1. In the application's `Startup.cs` file, add a method `public void ConfigureServices(IServiceCollection services)`
1. Register the `IRequestIdFactory` service as a singleton: `services.AddSingleton<IRequestIdFactory, RequestIdFactory>();`
1. Register the `IRequestId` service as scoped: `services.AddScoped<IRequestId, RequestId>();`
1 The `ConfigureServices` method should now look something like this:
  ``` C#
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton<IRequestIdFactory, RequestIdFactory>();
    services.AddScoped<IRequestId, RequestId>();
}
  ```

## Create and add a middleware that logs the request ID
1. Create a new folder in the application `Middleware`
1. In the folder, create a class `RequestIdMiddleware`
1. Create a constructor `public RequestIdMiddleware(RequestDelegate next, IRequestId requestId, ILogger<RequestIdMiddleware> logger)` and store the parameters in private fields
1. Add a method `public Task Invoke(HttpContext context)` and in its body log the request ID using the `ILogger` and `IRequestId` injected from the constructor
1. Your middleware class should look something like this:
  ``` C#
  public class RequestIdMiddleware
  {
      private readonly RequestDelegate _next;
      private readonly ILogger<RequestIdMiddleware> _logger;
  
      public RequestIdMiddleware(RequestDelegate next, ILogger<RequestIdMiddleware> logger)
      {
          _next = next;
          _logger = logger;
      }
  
      public Task Invoke(HttpContext context, IRequestId requestId)
      {
          _logger.LogInformation($"Request {requestId.Id} executing.");
  
          return _next(context);
      }
  }
  ```

1. Add the middleware to your pipeline back in `Startup.cs` by calling `app.UseMiddleware<RequestIdMiddleware>();`

## Configure a logger so you can see the request ID messages
1. In the `Startup.cs` file, add a parameter to the `Configure` method: `ILoggerFactory loggerFactory`
1. Add the `DebugLoggerProvider` at the beginning of the method body: `loggerFactory.AddDebug();`
1. Debug the application (F5) and you should see the messages containing the request ID being logged into the debug output

# Adding a unit test project

## Adding the xUnit myget feed to the application's feed configuration
1. Open the `nuget.config` file in the solution directory
1. Add the xUnit CI feed: `<add key="xUnit" value="https://www.myget.org/F/xunit/api/v2" />`

## Creating the test project
1. Create a new sub-directory of the solution directory `test` and add it to the solution as a solution folder
1. Create a new project in the `test` directory called `ProjectTest` using the "Class Library (Package)" project template
1. Delete the `Class1.cs` file
1. Open the `project.json` file and add a reference to `xunit` and `xunit.runner.dnx`, version "2.1.0-*" for both and ensure they restore successfully
1. Add a new class `MyTest.cs`
1. Add a method `public void MyFirstTest()` and decorate it with the `[Fact]` attribute, importing any required namespaces as you go
1. Do a simple assertion in the method body that will pass, e.g. `Assert.Equal(1, 1);`
1. Open the "Test Explorer" window
1. Build the solution and your tests should appear
