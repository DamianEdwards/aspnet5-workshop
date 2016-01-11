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
