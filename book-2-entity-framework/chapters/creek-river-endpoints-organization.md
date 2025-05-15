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
        // GET /api/campsites - Get all campsites
        app.MapGet("/api/campsites", (CreekRiverDbContext db) =>
        {
            return db.Campsites.ToList();
        });

        // GET /api/campsites/{id} - Get a specific campsite by ID
        app.MapGet("/api/campsites/{id}", (CreekRiverDbContext db, int id) =>
        {
            var campsite = db.Campsites
                .Include(c => c.CampsiteType)
                .SingleOrDefault(c => c.Id == id);

            if (campsite == null)
            {
                return Results.NotFound();
            }

            return Results.Ok(campsite);
        });

        // POST /api/campsites - Create a new campsite
        app.MapPost("/api/campsites", (CreekRiverDbContext db, Campsite campsite) =>
        {
            db.Campsites.Add(campsite);
            db.SaveChanges();
            return Results.Created($"/api/campsites/{campsite.Id}", campsite);
        });

        // PUT /api/campsites/{id} - Update a campsite
        app.MapPut("/api/campsites/{id}", (CreekRiverDbContext db, int id, Campsite campsite) =>
        {
            var existingCampsite = db.Campsites.Find(id);
            if (existingCampsite == null)
            {
                return Results.NotFound();
            }

            existingCampsite.Nickname = campsite.Nickname;
            existingCampsite.ImageUrl = campsite.ImageUrl;
            existingCampsite.CampsiteTypeId = campsite.CampsiteTypeId;

            db.SaveChanges();
            return Results.NoContent();
        });

        // DELETE /api/campsites/{id} - Delete a campsite
        app.MapDelete("/api/campsites/{id}", (CreekRiverDbContext db, int id) =>
        {
            var campsite = db.Campsites.Find(id);
            if (campsite == null)
            {
                return Results.NotFound();
            }

            db.Campsites.Remove(campsite);
            db.SaveChanges();
            return Results.NoContent();
        });
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
        // GET /api/reservations - Get all reservations
        app.MapGet("/api/reservations", (CreekRiverDbContext db) =>
        {
            return db.Reservations
                .Include(r => r.Campsite)
                .ThenInclude(c => c.CampsiteType)
                .ToList();
        });

        // GET /api/reservations/{id} - Get a specific reservation by ID
        app.MapGet("/api/reservations/{id}", (CreekRiverDbContext db, int id) =>
        {
            var reservation = db.Reservations
                .Include(r => r.Campsite)
                .ThenInclude(c => c.CampsiteType)
                .SingleOrDefault(r => r.Id == id);

            if (reservation == null)
            {
                return Results.NotFound();
            }

            return Results.Ok(reservation);
        });

        // POST /api/reservations - Create a new reservation
        app.MapPost("/api/reservations", (CreekRiverDbContext db, Reservation reservation) =>
        {
            // Calculate the total cost based on the campsite type fee and number of days
            var campsite = db.Campsites
                .Include(c => c.CampsiteType)
                .FirstOrDefault(c => c.Id == reservation.CampsiteId);

            if (campsite == null)
            {
                return Results.NotFound("Campsite not found");
            }

            // Calculate the number of days
            var numberOfDays = (reservation.CheckoutDate - reservation.CheckinDate).Days;

            // Set the total cost
            reservation.TotalCost = campsite.CampsiteType.FeePerNight * numberOfDays;

            db.Reservations.Add(reservation);
            db.SaveChanges();
            return Results.Created($"/api/reservations/{reservation.Id}", reservation);
        });

        // DELETE /api/reservations/{id} - Delete a reservation
        app.MapDelete("/api/reservations/{id}", (CreekRiverDbContext db, int id) =>
        {
            var reservation = db.Reservations.Find(id);
            if (reservation == null)
            {
                return Results.NotFound();
            }

            db.Reservations.Remove(reservation);
            db.SaveChanges();
            return Results.NoContent();
        });
    }
}
```

## Updating Program.cs

With our endpoint classes in place, we can now update `Program.cs` to use them. The updated file should look like this:

```csharp
using CreekRiver.Models;
using CreekRiver.Endpoints;
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

// Map API endpoints by resource
app.MapCampsiteEndpoints();
app.MapReservationEndpoints();

app.Run();
```

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