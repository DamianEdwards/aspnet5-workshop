# Build a simple API

## Setting up the MVC API project

1. Use the instructions in Getting Started to setup an Empty Web Application.
2. Add `Microsoft.AspNet.Mvc.Core` to `project.json`:

  ```JSON
  "dependencies": {
    "Microsoft.AspNetCore.Server.Kestrel": "1.0.0-*",
    "Microsoft.AspNetCore.Mvc.Core": "6.0.0-*",
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
      app.UseMvc();
  }
  ```

4. Create a folder called `Models` and create a class called `Product` in that folder:

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

## Returning JSON from the controller

1. Add the `Microsoft.AspNet.Mvc.Formatters.Json` to `project.json`:

  ```JSON
  "dependencies": {
    "Microsoft.AspNetCore.Server.Kestrel": "1.0.0-*",
    "Microsoft.AspNetCore.Mvc.Core": "6.0.0-*",
    "Microsoft.AspNetCore.Mvc.Formatters.Json": "6.0.0-*"
  },
  ```

2. Configure MVC to use the JSON formatter by changing the `ConfigureServices` in `Startup.cs` to use the following:

  ```C#
  public void ConfigureServices(IServiceCollection services)
  {
      services.AddMvcCore()
          .AddJsonFormatters();
  }
  ```

3. Add a static list of projects to the `ProductsController`:

  ```C#
  public class ProductsController
  {
      private static List<Product> _products = new List<Product>(new[] {
          new Product() { Id = 1, Name = "Computer" },
          new Product() { Id = 2, Name = "Radio" },
          new Product() { Id = 3, Name = "Apple" },
      });
      ...
  }
  
  ```
4. Change the `Get` method in `ProductsController` to return `IEnumerable<Product>` and return the list of products.

  ```C#
  public IEnumerable<Product> Get()
  {
      return _products;
  }
  ```

5. Run the application and navigate to `/api/products`. You should see a JSON payload of with all of the products.
6. Make a POST request to `/api/products`, you should see a JSON payload with all products.
7. Restrict the `Get` method to respond to GET requests by adding an `[HttpGet]` attribute to the method:

  ```C#
  [HttpGet]
  public IEnumerable<Product> Get()
  {
      return _products;
  }
  ```

## Add a method to Get a single product

1. Add a `Get` method to the `ProductsController` that takes an `int id` parameter and returns `Product`.

  ```C#
  public Product Get(int id)
  {
      return _products.FirstOrDefault(p => p.Id == id);
  }
  ```

2. Add an `HttpGet` route specifying the `id` as a route parameter:

  ```C#
  [HttpGet("{id}")]
  public Product Get(int id)
  {
      return _products.FirstOrDefault(p => p.Id == id);
  }
  ```

3. Run the application, and navigate to `/api/products/1`, you should see a JSON response for the first product.
4. Navigate to `/api/products/25`, it should return a 204 status code.
5. Change the `Get` method in the `ProductsController` to return a 404 if the product search returns null.

  ```C#
  [HttpGet("{id}")]
  public IActionResult Get(int id)
  {
      var product = _products.FirstOrDefault(p => p.Id == id);

      if (product == null)
      {
          return new HttpNotFoundResult();
      }

      return new HttpOkObjectResult(product);
  }
  ```
6. Run the application and navigate to `/api/products/40` and it should return a 404 status code.

## Adding to the list of products

1. Add a `Post` method to `ProductsController` the takes a `Product` as input and adds it to the list of products:

  ```C#
  public void Post(Product product)
  {
      _products.Add(product);
  }
  ```

2. Add an `[HttpPost]` attribute to the method to constrain it to the POST HTTP verb:

  ```C#
  [HttpPost]
  public void Post(Product product)
  {
      _products.Add(product);
  }
  ```
  
3. Add a `[FromBody]` to the `product` argument:

  ```C#
  [HttpPost]
  public void Post([FromBody]Product product)
  {
      _products.Add(product);
  }
  ```

4. Run the application, open a tool like [fiddler](http://www.telerik.com/fiddler) or [POSTman](https://www.getpostman.com/), and post a JSON payload with the `Content-Type` header `application/json` to `/api/products`:

  ```JSON
  {
     "Id" : "4",
     "Name": "4K Television"
  }
  ```

5. Make a GET request to `/api/products` and the new entity should show up in the list.

## Adding XML support

1. Add the `Microsoft.AspNet.Mvc.Formatters.Xml` package to `project.json`:

  ```JSON
  "dependencies": {
    "Microsoft.AspNetCore.Server.Kestrel": "1.0.0-*",
    "Microsoft.AspNetCore.Mvc.Core": "6.0.0-*",
    "Microsoft.AspNetCore.Mvc.Formatters.Json": "6.0.0-*",
    "Microsoft.AspNetCore.Mvc.Formatters.Xml": "6.0.0-*"
  },
  ```

2. In `Startup.cs` add a call to  `AddXmlDataContractSerializerFormatters()` chained off the `AddMvcCore` method in `ConfigureServices`:

  ```C#
  public void ConfigureServices(IServiceCollection services)
  {
      services.AddMvcCore()
          .AddJsonFormatters()
          .AddXmlDataContractSerializerFormatters();
  }
  ```
3. Run the application and make a request to `/api/products` with the accept header `application/xml`. The response should be an XML payload of the products.

## Restrict the ProductsController to be JSON only

1. Add a `[Produces("application/json")]` attribute to the `ProductsController` class:

  ```C#
  [Produces("application/json")]
  [Route("/api/products")]
  public class ProductsController
  ```
