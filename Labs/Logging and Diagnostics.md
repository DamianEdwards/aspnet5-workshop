## Logging

## Setting up your application for logging
1. Use the application from the first lab (or setup steps from the fisrt lab).
2. Add the console logging provider to `project.json`:

    ```JSON
      "dependencies": {
        "Microsoft.AspNet.IISPlatformHandler": "1.0.0-*",
        "Microsoft.AspNet.Server.Kestrel": "1.0.0-*",
        "Microsoft.Extensions.Logging.Console": "1.0.0-*"
      },
    ```
3. Navigate to `Startup.cs` Change the `Configure` method to:
    
    ```C#
        public void Configure(IApplicationBuilder app, ILoggerFactory loggerFactory)
        {
            loggerFactory.AddConsole();

            var startupLogger = loggerFactory.CreateLogger<Startup>();
            ...
        }
    ```

4. Add a log statement to the end of the `Configure` method:
    
    ```C#
    public void Configure(IApplicationBuilder app, ILoggerFactory loggerFactory)
    {
        ...
        startupLogger.LogInformation("Application startup complete!");
    }
    ```
    
5. In Visual Studio, change the active command to web by navigating to the play/run button and changing the drop down to the web command. 

    ![image](https://cloud.githubusercontent.com/assets/95136/12222924/633ef134-b7c1-11e5-9146-da36013da8d8.png)

6. Run the application and open a browser window with `http://localhost:5000/` as the address. You should see the default request logging in the framework as well as your custom log message.

## Filtering logs
1. Add a couple more loggging statements to the `Configure` method:
    
    ```C#
    public void Configure(IApplicationBuilder app, ILoggerFactory loggerFactory)
    {
        ...
        startupLogger.LogInformation("Application startup complete!");

        startupLogger.LogCritical("This is a critical message");
        startupLogger.LogDebug("This is a debug message");
        startupLogger.LogWarning("This is a warning message");
        startupLogger.LogError("This is an error message");
    }
    ```
    
2. Change the minimum log level for the console logger in `Startup.cs`:

    ```C#
    public void Configure(IApplicationBuilder app, ILoggerFactory loggerFactory)
    {
        loggerFactory.AddConsole(LogLevel.Debug);
        ...
    }
    ```

3. Run the application and open a browser window with `http://localhost:5000/` as the address. You should see more verbose logging from the framework and startup including debug messages.

4. Change the application to only show logs from the Startup category:

    ```C#
    public void Configure(IApplicationBuilder app, ILoggerFactory loggerFactory)
    {
        loggerFactory.AddConsole((category, level) => category == typeof(Startup).FullName);
        ...
    }
    ```

5. Run the application and open a browser window with `http://localhost:5000/` as the address. You should only logs written by the Startup logger.

## Adding other logging providers
1. Add the Serilog logger provider to `project.json`:

    ```JSON
      "dependencies": {
        "Microsoft.AspNet.IISPlatformHandler": "1.0.0-*",
        "Microsoft.AspNet.Server.Kestrel": "1.0.0-*",
        "Microsoft.Extensions.Logging.Console": "1.0.0-*",
        "Serilog.Framework.Logging": "1.0.0-*"
      },
    ```
2. Configure the Serilog in `Startup.cs` to write to a file called `logfile.txt` in the project root:
    
    ```C#
    public class Startup
    {
        public Startup(IApplicationEnvironment appEnv)
        {
            var logFile = Path.Combine(appEnv.ApplicationBasePath, "logfile.txt");
    
            Log.Logger = new LoggerConfiguration()
                            .WriteTo.TextWriter(File.CreateText(logFile))
                            .CreateLogger();
        }
    }
    ```
3. Add the Serilog provider in `Configure`:

    ```C#
    public void Configure(IApplicationBuilder app, ILoggerFactory loggerFactory)
    {
        loggerFactory.AddConsole();
        loggerFactory.AddSerilog();
        ...
    }
    ```

4. Run the application and open a browser window with `http://localhost:5000/` as the address. You should only a file called `logfile.txt` appear in your application root. 

5. Closing the conosle window and open the file, the application logs should be in there.

## Extra 
1. Try adding more advanced filters with different levels.
2. Try configuring logging using the Configuration system (`IConfiguration`).

# Diagnostic pages

## Add a middleware to the above application that throws an exception. Your Configure method should look something like this:

    ```C#
    public void Configure(IApplicationBuilder app, ILoggerFactory loggerFactory)
    {
        loggerFactory.AddConsole();

        loggerFactory.AddSerilog();

        var startupLogger = loggerFactory.CreateLogger<Startup>();

        app.UseIISPlatformHandler();

        app.Run((context) =>
        {
            throw new InvalidOperationException("Oops!");
        });

        startupLogger.LogInformation("Application startup complete!");
    }
    ```
