# Organizing Endpoints by Resource

In this chapter, we'll learn how to organize our API endpoints by resource using extension methods. This approach helps keep our `Program.cs` file clean and makes our code more maintainable as the application grows.

## Understanding the Endpoints Directory Structure

In a well-organized API project, it's a good practice to separate endpoints by resource. We'll create an `Endpoints` directory with a separate file for each resource type:

```
CreekRiver/
├── Endpoints/
│   ├── CampsiteEndpoints.cs
│   └── ReservationEndpoints.cs
├── Models/
│   ├── Campsite.cs
│   ├── CampsiteType.cs
│   └── Reservation.cs
├── Program.cs
└── ...
```

This structure makes it easy to find and modify endpoints related to a specific resource.

## Creating the CampsiteEndpoints Class

Let's create our first endpoint class for campsites. Create a new file called `Endpoints/CampsiteEndpoints.cs`:

```csharp
using CreekRiver.Models;
using Microsoft.EntityFrameworkCore;

namespace CreekRiver.Endpoints;

public static class CampsiteEndpoints
{
    public static void MapCampsiteEndpoints(this WebApplication app)
    {
        // All of the endpoints relevent to campsites will be here
    }
}
```

## Understanding Extension Methods

The `MapCampsiteEndpoints` method is an extension method for the `WebApplication` class. Extension methods allow you to add methods to existing types without modifying them.

The `this` keyword before the first parameter indicates that this is an extension method. When you call `app.MapCampsiteEndpoints()` in `Program.cs`, you're actually calling this extension method.

## Creating the ReservationEndpoints Class

Now, let's create a similar class for reservation endpoints. Create a new file called `Endpoints/ReservationEndpoints.cs`:

```csharp
using CreekRiver.Models;
using Microsoft.EntityFrameworkCore;

namespace CreekRiver.Endpoints;

public static class ReservationEndpoints
{
    public static void MapReservationEndpoints(this WebApplication app)
    {
        // All of the endpoints relevent to reservations will be here
    }
}
```

## Updating Program.cs

With our endpoint classes in place, we can now update `Program.cs` to use them. Your complete `Program.cs` file should look like this:

```csharp
using Microsoft.EntityFrameworkCore;
using CreekRiver.Endpoints;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.

// Add the DbContext to the container
builder.Services.AddDbContext<CreekRiverDbContext>(options =>
    options.UseNpgsql(builder.Configuration.GetConnectionString("CreekRiverDbConnectionString")));

var app = builder.Build();

// Map API endpoints by resource
app.MapCampsiteEndpoints();
app.MapReservationEndpoints();

app.Run();
```

Key points about this setup:

1. **Using Directives**: We include `CreekRiver.Endpoints` to access our endpoint extension methods.

2. **Extension Method Calls**: Before `app.Run()`, we call our extension methods to register all endpoints for each resource.

3. **Clean Organization**: The `Program.cs` file remains clean and focused on configuration, while the actual endpoint definitions are organized in separate files by resource.

In the following chapters, you will define the specific endpoints one at a time within each endpoint class.

## Benefits of This Approach

Organizing endpoints by resource offers several benefits:

1. **Better organization** - Endpoints are grouped by resource, making it easier to find and modify them.
2. **Improved maintainability** - Each resource's endpoints are contained in a single file, reducing the complexity of `Program.cs`.
3. **Scalability** - As your application grows, you can add new endpoint classes without cluttering `Program.cs`.
4. **Testability** - Endpoint classes can be tested independently.
5. **Reusability** - Extension methods can be reused across different applications.

## Conclusion

In this chapter, we've learned how to organize our API endpoints by resource using extension methods. This approach helps keep our code clean, maintainable, and scalable as our application grows.

In the next chapters, we'll continue building our API by implementing specific endpoints for campsites and reservations.

Up Next: [Creating the first endpoint that will GET campsites](./creek-river-get-campsites.md)

## 🔍 Additional Materials

- [Extension Methods in C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/extension-methods)
- [ASP.NET Core Minimal APIs](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis)
- [API Design Best Practices](https://docs.microsoft.com/en-us/azure/architecture/best-practices/api-design)