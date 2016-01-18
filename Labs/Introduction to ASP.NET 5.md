
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
  
1. Create a new `WebHostBuilder` in the `Program.cs` file to setup the server, urls and Startup class:

  ```C#
  public static void Main(string[] args)
  {
      var host = new WebHostBuilder()
                  .UseServer("Microsoft.AspNet.Server.Kestrel")
                  .UseUrls("http://localhost:5001")
                  .UseStartup<Startup>()
                  .Build();
      host.Run();
  }
  ```
  
1. Run the application and open a browser to `http://localhost:5001/`, you should see hello world printed.

## Changing the environment to Development

1. Change the `Main` in `Program.cs` to set the environment to Development:

  ```C#
  public static void Main(string[] args)
  {
      var host = new WebHostBuilder()
                  .UseServer("Microsoft.AspNet.Server.Kestrel")
                  .UseUrls("http://localhost:5001")
                  .UseEnvironment(EnvironmentName.Development)
                  .UseStartup<Startup>()
                  .Build();
      host.Run();
  }
  ```
  
2. Run the application and it should now be running with the "Development" environment.

## Reading the default hosting configuration

1. Add a file called `hosting.json` in the project directory with the following contents:

  ```JSON
  {
    "server": "Microsoft.AspNet.Server.Kestrel",
    "server.urls": "http://localhost:5001"
  }
  ```

1. Change the `Main` method in `Program.cs` to read the default hosting configuration:

  ```C#
  public static void Main(string[] args)
  {
      var host = new WebApplicationBuilder()
                  .UseDefaultConfiguration(args)
                  .UseStartup<Startup>()
                  .Build();
      host.Run();
  }
  ```
1. Run the application, it should have the same settings that it had previously instead they should be read from the `hosting.json` file.

1. Change the URL in `hosting.json` to `http://localhost:3000`:

1. Run the application on `http://localhost:3000`.

## Setting the environment using environment variables

1. Now you're going to edit the launch profile so that it includes an environment variable that will override the message from config.json

1. Right-mouse click on the project and select "Properties"
1. Open the "Debug" tab
1. Add an environment variable named "ASPNET_ENVIRONMENT" with a value of "Development"
1. Run the application again and the current environment should be set to Development.

## Serving static files

1. Add the `Microsoft.AspNet.StaticFiles` package to `project.json`:

  ```JSON
  "dependencies": {
    "Microsoft.AspNet.Server.Kestrel": "1.0.0-*",
    "Microsoft.AspNet.StaticFiles": "1.0.0-*"
  },
  ```
1. Go to `Startup.cs` in the `Configure` method and add `UseStaticFiles` before the hello world middleware:

  ```C#
  public void Configure(IApplicationBuilder app)
  {
      app.UseStaticFiles();

      app.Run(async context =>
      {
          await context.Response.WriteAsync("Hello World");
      });
  }
  ```
  
1. Create a folder called `wwwroot` in the project folder.
1. Create a file called `index.html` with the following contents in the `wwwroot` folder:

  ```
  <!DOCTYPE html>
  <html>
  <head>
      <meta charset="utf-8" />
      <title></title>
  </head>
  <body>
      <h1>Hello from ASP.NET 5!</h1> 
  </body>
  </html>
  ```

1. Run the application and navigate to the root. It should show the hello world middleware.
1. Navigate to `index.html` and it should show the static page in `wwwroot`.

## Adding default document support

1. Change the static files middleware in `Startup.cs` from `app.UseStaticFiles()` to `app.UseFileServer()`.
2. Run the application. The default page `index.html` should show when navigating to the root of the site.

## Add IIS Support

1. Add the `Microsoft.AspNet.IISPlatformHandler` package to `project.json`:

  ```JSON
  "dependencies": {
    "Microsoft.AspNet.IISPlatformHandler": "1.0.0-*",
    "Microsoft.AspNet.Server.Kestrel": "1.0.0-*",
    "Microsoft.AspNet.StaticFiles": "1.0.0-*"
  },
  ```
1. Add `UseIISPlatformHandlerUrl()` to the `Main` method in `Program.cs`:

  ```C#
  public static void Main(string[] args)
  {
      var host = new WebApplicationBuilder()
                  .UseDefaultConfiguration(args)
                  .UseIISPlatformHandlerUrl()
                  .UseStartup<Startup>()
                  .Build();
      host.Run();
  }
  ```

1. Add `app.UseIISPlatformHandler()` to the `Configure` method in `Startup.cs`:

  ```C#
  public void Configure(IApplicationBuilder app)
  {
      app.UseIISPlatformHandler();
      app.UseFileServer();

      ...
  }
  ```

1. Right click project properties, go to the `Debug` tab and click `New...` next to the profile:

    ![image](https://cloud.githubusercontent.com/assets/95136/12227671/00b77b96-b828-11e5-97ef-487cac89fb3f.png)

1. Save the launch profile called IIS Express navigate to Properties folder and change `launchSettings.json` to the following:

  ```JSON
  {
    "iisSettings": {
      "windowsAuthentication": false,
      "anonymousAuthentication": true,
      "iisExpress": {
        "applicationUrl": "http://localhost:18885/",
        "sslPort": 0
      }
    },
    "profiles": {
      "IIS Express": {
        "commandName": "IISExpress",
        "launchBrowser": true,
        "environmentVariables": {
          "ASPNET_ENVIRONMENT": "Development"
        }
      }
    }
  }
  ```
1. In `project.json` add a `web` command that points to the application name:

  ```JSON
  "commands": {
    "{ApplicationName}": "{ApplicationName}",
    "web": "{ApplicationName}"
  },
  ```

1. Run the application. It should be running on IIS Express using the Http Platform handler. There should be a `web.config` generated by visual studio in the `wwwroot` folder.

## Capturing standard output from application when running in IIS/IIS Express

1. Open web.config and add the `stdoutLogEnabled="true"` to the `httpPlatform` section:

  ```XML
  <httpPlatform processPath="%DNX_PATH%" arguments="%DNX_ARGS%" forwardWindowsAuthToken="false" stdoutLogEnabled="true" startupTimeLimit="3600" />
  ```

1. Run the application. There should be a log file created in the application root.
