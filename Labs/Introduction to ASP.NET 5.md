# Creating an inline middleware
1. Start with the application you created in the first lab, or just create a new empty ASP.NET 5 application
1. Open `Startup.cs`
1. Create an inline middleware that runs **before** the hello world delegate that sets the culture for the current request from the query string:
  
  ``` C#
  public void Configure(IApplicationBuilder app)
  {
    app.Use((context, next) =>
    {
      var cultureQuery = context.Request.Query["culture"];
      if (!string.IsNullOrWhiteSpace(cultureQuery))
      {
        var culture = new CultureInfo(cultureQuery);
  #if !DNXCORE50
        Thread.CurrentThread.CurrentCulture = culture;
        Thread.CurrentThread.CurrentUICulture = culture;
  #else
        CultureInfo.CurrentCulture = culture;
        CultureInfo.CurrentUICulture = culture;
  #endif
      }
      
      // Call the next delegate/middleware in the pipeline
      return next();
    });
    
    app.Run(async (context) =>
    {
        await context.Response.WriteAsync($"Hello {CultureInfo.CurrentCulture.DisplayName}");
    });
  }
  ```
  
1. Run the app now and set the culture via the query string, e.g. http://localhost/?culture=no

# Add configuration
1. Add a constructor to the application's `Startup.cs`
1. Create a new `Configuration` object in the constructor and assign it to a new private class field `IConfiguration _configuration`
1. Add a reference to the `Microsoft.Framework.ConfigurationModel.Json` package in the application's `project.json` file
1. Back in the `Startup.cs`, add a call to `.AddJsonFile("config.json")` immediately after the creation of the `Configuration` object (inline, chained method). It should now look like the following:

  ``` C#
  public class Startup
  {
    private readonly IConfiguration _configuration;

    public Startup()
    {
        var configuration = new Configuration()
            .AddJsonFile("config.json");

        _configuration = configuration;
    }
    ...
  }
  ```
  
1. Add a new JSON file to the project called `config.json`
1. Add a new key/value pair to the `config.json` file: `"message": "Hello from config.json!"`
1. Change the code in `Startup.cs` to use the message from the configuration system:

  ``` C#
  app.Run(async (context) =>
  {
      await context.Response.WriteAsync(_configuration["message"]);
  });
  ```
  
1. Run the application and the message from `config.json` should be returned
1. Change the message in the `config.json` file and refresh the page (without changing any other code). Note that the message hasn't changed as the configuration was only read when the application was started.
1. Go back to Visual Studio and touch and save the `Startup.cs` file to force the process to restart
1. Go back to the browser now and refresh the page and it should show the updated message

## Override configuration from environment variables
1. In the application's `Startup.cs` file, add a call to `.AddEnvironmentVariables()` immediately after the call to `.AddJsonFile`
  - Note that the order you chain these methods together matters, as values in later configuration sources override matching values from earlier sources
1. Your constructor should now look like this:
  
  ``` C#
  public Startup()
  {
      var configuration = new Configuration()
          .AddJsonFile("config.json")
          .AddEnvironmentVariables();

      _configuration = configuration;
  }
  ```
  
1. Now you're going to edit the launch profile so that it includes an environment variable that will override the message from `config.json`
1. Right-mouse click on the project and select "Properties"
1. Open the "Debug" tab
1. Add an environment variable named "message" with a value of "Hello from environment variable!"
1. Run the application again and the message should now be the value from the environment variable

# Configure the project to use packages from the unstable feed
1. Create a file called `nuget.config` to the solution directory (next to the .sln file)
1. Add configuration to bring in the ASP.NET 5 dev feed and nuget.org
  
  ``` xml
  <?xml version="1.0" encoding="utf-8"?>
  <configuration>
    <packageSources>
      <add key="AspNetVNext" value="https://www.myget.org/F/aspnetvnext/api/v2" />
      <add key="NuGet" value="https://nuget.org/api/v2/" />
    </packageSources>
  </configuration>
  ```
  
1. Configure the project to use the latest DNX version you have installed by opening the `global.json` file in the solution directory and setting the `sdk` property to that version.
  - Note, you can use DNVM at the command line to list the versions you have installed

  ``` JSON
  {
      "projects": [ "src", "test" ],
      "sdk": {
          "version": "1.0.0-beta6-12032"
      }
  }
  ```

1. Close and re-open the solution in Visual Studio
1. In the application's `project.json` file, replace all references to `beta4` with `*` to indicate you want the latest packages from the configured feeds, e.g.

  ``` JSON
  "dependencies": {
    "Microsoft.AspNet.IISPlatformHandler": "1.0.0-*",
    "Microsoft.AspNet.Server.Kestrel": "1.0.0-*",
    "Microsoft.Framework.ConfigurationModel.Json": "1.0.0-*"
  },
  ```
  
1. Ensure the packages have finished restoring without errors and that they're all from the same beta release before continuing
1. The application is now configured to use the latest packages from the unstable feed

# Move the middleware to its own type
1. Create a new class in the application `RequestCultureMiddleware`
1. Add a constructor that takes a parameter `RequestDelegate next` and assigns it to a private field `private readonly RequestDelegate _next`
1. Add a method `public Task Invoke(HttpContext context)`
1. Copy the code from the inline middleware delegate in the application's `Startup.cs` file to the `Invoke` method you just created and fix the `next` method name
1. Your middleware class should now look something like this:

  ``` C#
  public class RequestCultureMiddleware
  {
      private readonly RequestDelegate _next;
  
      public RequestCultureMiddleware(RequestDelegate next)
      {
          _next = next;
      }
  
      public Task Invoke(HttpContext context)
      {
          var cultureQuery = context.Request.Query["culture"];
          if (!string.IsNullOrWhiteSpace(cultureQuery))
          {
              var culture = new CultureInfo(cultureQuery);
  #if !DNXCORE50
              Thread.CurrentThread.CurrentCulture = culture;
              Thread.CurrentThread.CurrentUICulture = culture;
  #else
              CultureInfo.CurrentCulture = culture;
              CultureInfo.CurrentUICulture = culture;
  #endif
          }
  
          return _next(context);
      }
  }
  ```
  
1. Back in the application's `Startup.cs` file, delete the inline middleware delegate
1. Add your new middleware class to the HTTP pipeline:

  ``` C#
  app.UseMiddleware<RequestCultureMiddleware>();
  ```
  
1. Run the application again and see that the middleware is now running as a class
