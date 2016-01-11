
## Create a new Console project, configure it to use the unstable packages feed, and upgrade it to RC2

1. Open Visual Studio 2015
1. Create a new ASP.NET 5 application:
  1. File -> New -> ASP.NET Web Application -> ASP.NET 5 -> Console Application (Package)
1. Open the `global.json` file and remove the `sdk` section. This will instruct Visual Studio to use the version configured as "default" from the steps before.
1. Add a `NuGet.config` file to the root of the solution with the following contents. This will configure NuGet to use the unstable feed for packages:

   ``` xml
   <?xml version="1.0" encoding="utf-8"?>
   <configuration>
   <packageSources>
       <add key="AspNetCiDev" value="https://www.myget.org/F/aspnetcidev/api/v3/index.json" />
       <add key="NuGet" value="https://api.nuget.org/v3/index.json" />
   </packageSources>
   </configuration>
   ```
1. You should have a `Program.cs` with an empty `Main` method.

## Add ASP.NET 5 to the project
1. Add `Microsoft.AspNet.Server.Kestrel` to `project.json`.

  ```JSON
  "dependencies": {
    "Microsoft.AspNet.Server.Kestrel": "1.0.0-*"
  },
  ```

1. Add a `Startup.cs` file with a `Configure` that prints out the string "Hello World" (you may need to add `using Microsoft.AspNet.Http` for the WriteAsync extension method):

  ```C#
  public class Startup
  {
      public void Configure(IApplicationBuilder app)
      {
          app.Run(async context =>
          {
              await context.Response.WriteAsync("Hello World");
          });
      }
  }
  ```
  
1. Create a new `WebApplicationBuilder` to setup the server, urls and Startup class:

  ```C#
  public static void Main(string[] args)
  {
      var app = new WebApplicationBuilder()
                  .UseServer("Microsoft.AspNet.Server.Kestrel")
                  .UseUrls("http://localhost:5001")
                  .UseStartup<Startup>()
                  .Build();
      app.Run();
  }
  ```
  
1. Run the application and open a browser to `http://localhost:5001/`, you should see hello world printed.
