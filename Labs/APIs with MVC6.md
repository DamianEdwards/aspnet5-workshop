# Build a simple API

## Setting up the MVC API project

1. Use the instructions in Getting Started to setup an Empty Web Application.
2. Add `Microsoft.AspNet.Mvc.Core` to `project.json`:

  ```JSON
    "dependencies": {
    "Microsoft.AspNet.IISPlatformHandler": "1.0.0-*",
    "Microsoft.AspNet.Server.Kestrel": "1.0.0-*",
    "Microsoft.AspNet.Mvc.Core": "6.0.0-*",
  },
  ```
3. In `Startup.cs` add `services.AddMvcCore()` to `ConfigureServices` and add `app.UseMvc()` to `Configure`:
  
  ```C#
  public void ConfigureServices(IServiceCollection services)
  {
      services.AddMvcCore();
  }
  
  public void Configure(IApplicationBuilder app)
  {
      app.UseIISPlatformHandler();

      app.UseMvc();
  }
  ```

4. Create a folder called `Modles` and create a class called `Product` in that folder:

  ```C#
  public class Product
  {
      public int Id { get; set; }
      public string Name { get; set; }
  }
  ```
  
5. Create a folder called `Controllers` and create a class called `ProductsController` in that folder.
6. Add an attribute route `[Route("/api/products")]` to the `ProductsController` class:

  ```C#
  [Route("/api/products")]
  public class ProductsController
  {
  }
  ```
  
7. Add a `Get` method to `ProductsController` that returns a `string` "Hello API World" with an attribute route

  ```C#
  [Route("/api/products")]
  public class ProductsController
  {
    public string Get() => "Hello World";
  }
  ```

8. Run the application and navigate to `/api/products`, it should return the string "Hello World".
