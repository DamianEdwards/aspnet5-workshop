# Getting Started

## Installing & updating ASP.NET 5
1. Go to https://docs.asp.net/en/latest/getting-started/index.html and follow the instructions for installing ASP.NET 5 on your OS.
1. Go to a command prompt and update `dnvm`:

   ```
   dnvm update-self
   ```
1. From a command prompt, upgrade to the latest DNX from the unstable feed:

   ```
   dnvm upgrade -u
   ```
1. List out your installed DNX versions to ensure the above steps worked and that the newest DNX version is marked as "default" and is highlighted for use:

   ```
   dnvm list
   ```

## Create a new project, configure it to use the unstable packages feed, and upgrade it to RC2
1. Open Visual Studio 2015
1. Create a new ASP.NET 5 application:
  1. File -> New -> ASP.NET Web Application -> ASP.NET 5 -> Empty Web Application
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
1. Open the `project.json` file and change the versions listed in the `dependencies` section from "1.0.0-rc1-final" to "1.0.0-*". This will instruct NuGet to download the latest builds of these packages from the feeds configured in the step above.
  1. Open the "References" node in Solution Explorer and confirm the package versions are "1.0.0-rc2-*"
1. Update the `commands` section of the `project.json` file to look like the following, ensuring you use the name of your application:
  
  ``` JSON
  "commands": {
    "WebApplication11": "WebApplication11",
    "web": "WebApplication11"
  },
  ```
1. Update the `public static void Main` method in `Startup.cs` to match the following:
  
  ``` c#
  public static void Main(string[] args)
  {
      var app = new WebApplicationBuilder()
          .UseServer("Microsoft.AspNet.Server.Kestrel")
          .UseConfiguration(WebApplicationConfiguration.GetDefault(args))
          .UseStartup<Startup>()
          .Build();

      app.Run();
  }
  ```
1. Verify your application builds and runs (Ctrl+F5) 
