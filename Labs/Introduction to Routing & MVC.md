# Introduction to Routing & MVC

## Install the routing package
1. Use the application you created in the first lab
1. Open the `project.json` file
1. In the `dependencies` section, add an entry for the "Microosft.AspNet.Routing.Extensions" package:
  ``` JSON
  "dependencies": {
    "Microsoft.AspNet.IISPlatformHandler": "1.0.0-*",
    "Microsoft.AspNet.Server.Kestrel": "1.0.0-*",
    "Microsoft.AspNet.Routing.Extensions": "1.0.0-*"
  }
  ```
1. Open the `Startup.cs` file
1. In the `Configure` method, create a `RouteBuilder` with a handler for the root of the site and add it to the middleware pipeline:
  ``` c#
  public void Configure(IApplicationBuilder app)
  {
      app.UseIISPlatformHandler();

      var routeBuilder = new RouteBuilder(app);

      routeBuilder.MapGet("", context => context.Response.WriteAsync("Hello from Routing!"));
            
      app.UseRouter(routeBuilder.Build());
  }
  ```
1. Run the site and verify your middleware is hit via routing (Ctrl+F5)

## Capture and use route data
1. Add another route that captures the name of an item from the URL, e.g. "item/{itemName}", and displays it in the response:
  ``` c#
  routeBuilder.MapGet("item/{itemName}", context => context.Response.WriteAsync($"Item: {context.GetRouteValue("itemName")}"));
  ```
1. Run the site and verify that your new route works
1. Modify the route to include a route constraint on the captured segmet, enforcing it to be a number:
  ``` c#
  routeBuilder.MapGet("item/{id:int}", context => context.Response.WriteAsync($"Item ID: {context.GetRouteValue("id")}"));
  ```
1. Run the site again and see that the route is only matched when the captured segment is a valid number
1. Modify the router to include both versions of the route above (with and without the route constraint)
1. Experiment with changing the order the routes are added and observe what affect that has on which route is matched for a given URL

