# Updating `Program.cs` to use all required EF packages

In this chapter, we'll update our `Program.cs` file to configure our API to use Entity Framework Core and set up the necessary services for our application.

## Understanding Program.cs in Minimal APIs

In ASP.NET Core minimal APIs, the `Program.cs` file is the entry point of the application. It contains all the configuration code needed to set up the web server, middleware, and services.

The minimal API approach simplifies the startup process by eliminating the need for separate `Startup.cs` and `Program.cs` files, which were common in previous versions of ASP.NET Core.

## Updating Program.cs

Let's update our `Program.cs` file to configure Entity Framework Core and set up our API:

1. Open the `Program.cs` file in your project.

2. First, add the necessary using directives at the top of the file:

```csharp
using CreekRiver.Models;
using Microsoft.EntityFrameworkCore;
using System.Text.Json.Serialization;
using Microsoft.AspNetCore.Http.Json;
```

3. Remove all of the weather forecast related code from `Program.cs` as we won't be using it.

4. Add the following code right above `var app = builder.Build();`:

```csharp
// Allows passing datetimes without time zone data
AppContext.SetSwitch("Npgsql.EnableLegacyTimestampBehavior", true);

// Allows our api endpoints to access the database through Entity Framework Core
builder.Services.AddNpgsql<CreekRiverDbContext>(builder.Configuration["CreekRiverDbConnectionString"]);

// Configure JSON serialization options
builder.Services.Configure<JsonOptions>(options =>
{
    // This tells the API to ignore circular references in JSON serialization
    options.SerializerOptions.ReferenceHandler = ReferenceHandler.IgnoreCycles;

    // This tells the API to use camel case for property names in JSON
    options.SerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase;

    // This tells the API to write Enums as strings in JSON
    options.SerializerOptions.Converters.Add(new JsonStringEnumConverter());
});
```

Let's break down what this code does:

- `AppContext.SetSwitch("Npgsql.EnableLegacyTimestampBehavior", true)`: This enables legacy timestamp behavior for Npgsql, which allows passing datetimes without time zone data.

- `builder.Services.AddNpgsql<CreekRiverDbContext>(...)`: This registers our `CreekRiverDbContext` with the dependency injection container and configures it to use Npgsql with the connection string from our configuration.

- `builder.Services.Configure<JsonOptions>(...)`: This configures JSON serialization options for our API:
  - `ReferenceHandler.IgnoreCycles`: This tells the JSON serializer to ignore circular references, which can occur when entities reference each other.
  - `JsonNamingPolicy.CamelCase`: This tells the JSON serializer to use camel case for property names in JSON (e.g., `firstName` instead of `FirstName`).
  - `JsonStringEnumConverter`: This tells the JSON serializer to write enums as strings instead of numbers.

## Understanding Dependency Injection

The line `builder.Services.AddNpgsql<CreekRiverDbContext>(...)` is an example of dependency injection, which is a design pattern that ASP.NET Core uses extensively.

Dependency Injection (DI) is a technique where an object receives other objects that it depends on, rather than creating them itself. In ASP.NET Core, the DI container is responsible for creating and managing these dependencies.

When we register our `CreekRiverDbContext` with the DI container, we're telling ASP.NET Core to create an instance of `CreekRiverDbContext` whenever a component (like an API endpoint) needs one.

## The Complete Program.cs File

Here's what your complete `Program.cs` file should look like after these changes:

```csharp
using CreekRiver.Models;
using Microsoft.EntityFrameworkCore;
using System.Text.Json.Serialization;
using Microsoft.AspNetCore.Http.Json;

var builder = WebApplication.CreateBuilder(args);

// Allows passing datetimes without time zone data
AppContext.SetSwitch("Npgsql.EnableLegacyTimestampBehavior", true);

// Allows our api endpoints to access the database through Entity Framework Core
builder.Services.AddNpgsql<CreekRiverDbContext>(builder.Configuration["CreekRiverDbConnectionString"]);

// Configure JSON serialization options
builder.Services.Configure<JsonOptions>(options =>
{
    // This tells the API to ignore circular references in JSON serialization
    options.SerializerOptions.ReferenceHandler = ReferenceHandler.IgnoreCycles;

    // This tells the API to use camel case for property names in JSON
    options.SerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase;

    // This tells the API to write Enums as strings in JSON
    options.SerializerOptions.Converters.Add(new JsonStringEnumConverter());
});

var app = builder.Build();

app.Run();
```

## Conclusion

In this chapter, we've updated our `Program.cs` file to configure Entity Framework Core and set up the necessary services for our application. We've learned about:

1. The role of `Program.cs` in minimal APIs
2. How to configure Entity Framework Core with Npgsql
3. How to configure JSON serialization options
4. The concept of dependency injection

In the next chapter, we'll create our first migration to create the database tables and seed the database with initial data.

Up Next: [Creating their first migration](./creek-river-migration.md)

## 🔍 Additional Materials

- [ASP.NET Core Minimal APIs](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis)
- [Dependency Injection in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection)
- [JSON serialization in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/web-api/advanced/formatting)
- [Entity Framework Core with ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/data/ef-rp/intro)