# Basic Features with DTOs

In this chapter, we'll implement the basic features for the Loncotes Library API using DTOs to shape our responses. We'll see how DTOs give us flexibility in how we present data to clients, and we'll use AutoMapper to simplify the mapping between our models and DTOs.

## Creating Endpoint Classes

First, let's create the endpoint classes that will organize our API endpoints by resource. This approach keeps our code clean and maintainable.

### Material Endpoints

Create a new file called `MaterialEndpoints.cs` in the `Endpoints` directory:

```csharp
// Endpoints/MaterialEndpoints.cs
using AutoMapper;
using LoncotesLibrary.Data;
using LoncotesLibrary.DTOs;
using LoncotesLibrary.Models;
using Microsoft.EntityFrameworkCore;

namespace LoncotesLibrary.Endpoints
{
    public static class MaterialEndpoints
    {
        public static void MapMaterialEndpoints(this WebApplication app)
        {
            app.MapGet("/materials", GetAllMaterials);
            app.MapGet("/materials/{id}", GetMaterialById);
            app.MapPost("/materials", CreateMaterial);
            app.MapPut("/materials/{id}/discontinue", RemoveMaterialFromCirculation);
        }

        private static IResult GetAllMaterials(LoncotesLibraryDbContext db, IMapper mapper)
        {
            // Basic implementation - return empty list for now
            return Results.Ok(new List<MaterialDto>());
        }

        private static IResult GetMaterialById(LoncotesLibraryDbContext db, IMapper mapper, int id)
        {
            // Basic implementation - return NotFound for now
            return Results.NotFound();
        }

        private static IResult CreateMaterial(LoncotesLibraryDbContext db, IMapper mapper, MaterialDto materialDto)
        {
            // Basic implementation - return a simple created response
            return Results.Created($"/materials/1", new MaterialDto { Id = 1 });
        }

        private static IResult RemoveMaterialFromCirculation(LoncotesLibraryDbContext db, int id)
        {
            // Basic implementation - return NoContent
            return Results.NoContent();
        }
    }
}
```

### Patron Endpoints

Create a new file called `PatronEndpoints.cs` in the `Endpoints` directory:

```csharp
// Endpoints/PatronEndpoints.cs
using AutoMapper;
using LoncotesLibrary.Data;
using LoncotesLibrary.DTOs;
using LoncotesLibrary.Models;
using Microsoft.EntityFrameworkCore;

namespace LoncotesLibrary.Endpoints
{
    public static class PatronEndpoints
    {
        public static void MapPatronEndpoints(this WebApplication app)
        {
            app.MapGet("/patrons", GetAllPatrons);
            app.MapGet("/patrons/{id}", GetPatronById);
            app.MapPut("/patrons/{id}", UpdatePatron);
            app.MapPut("/patrons/{id}/deactivate", DeactivatePatron);
        }

        private static IResult GetAllPatrons(LoncotesLibraryDbContext db, IMapper mapper)
        {
            // Basic implementation - return empty list for now
            return Results.Ok(new List<PatronDto>());
        }

        private static IResult GetPatronById(LoncotesLibraryDbContext db, IMapper mapper, int id)
        {
            // Basic implementation - return NotFound for now
            return Results.NotFound();
        }

        private static IResult UpdatePatron(LoncotesLibraryDbContext db, IMapper mapper, int id, PatronDto patronDto)
        {
            // Basic implementation - return NoContent
            return Results.NoContent();
        }

        private static IResult DeactivatePatron(LoncotesLibraryDbContext db, int id)
        {
            // Basic implementation - return NoContent
            return Results.NoContent();
        }
    }
}
```

### Checkout Endpoints

Create a new file called `CheckoutEndpoints.cs` in the `Endpoints` directory:

```csharp
// Endpoints/CheckoutEndpoints.cs
using AutoMapper;
using LoncotesLibrary.Data;
using LoncotesLibrary.DTOs;
using LoncotesLibrary.Models;
using Microsoft.EntityFrameworkCore;

namespace LoncotesLibrary.Endpoints
{
    public static class CheckoutEndpoints
    {
        public static void MapCheckoutEndpoints(this WebApplication app)
        {
            app.MapGet("/checkouts", GetAllCheckouts);
            app.MapPost("/checkouts", CreateCheckout);
            app.MapPut("/checkouts/{id}/return", ReturnCheckout);
        }

        private static IResult GetAllCheckouts(LoncotesLibraryDbContext db, IMapper mapper)
        {
            // Basic implementation - return empty list for now
            return Results.Ok(new List<CheckoutDto>());
        }

        private static IResult CreateCheckout(LoncotesLibraryDbContext db, IMapper mapper, CheckoutDto checkoutDto)
        {
            // Basic implementation - return a simple created response
            return Results.Created($"/checkouts/1", new CheckoutDto { Id = 1 });
        }

        private static IResult ReturnCheckout(LoncotesLibraryDbContext db, int id)
        {
            // Basic implementation - return NoContent
            return Results.NoContent();
        }
    }
}
```

## Update Program.cs to Map Endpoints

Now, let's update the `Program.cs` file to map our endpoints:

```csharp
using LoncotesLibrary.Endpoints; // Add this directive at the top

// ... existing code

// Map endpoints
MaterialEndpoints.MapMaterialEndpoints(app);
PatronEndpoints.MapPatronEndpoints(app);
CheckoutEndpoints.MapCheckoutEndpoints(app);

app.Run();
```

Now that we have set up our endpoint classes with basic implementations, we're ready to move on to implementing the full functionality with database queries and DTOs.

## Next Steps

In the next chapter, we'll implement the full functionality for our endpoints, including database queries and proper DTO mapping.

Up Next: [Implementing Endpoints with DTOs](./loncotes-dto-implementing-endpoints.md)