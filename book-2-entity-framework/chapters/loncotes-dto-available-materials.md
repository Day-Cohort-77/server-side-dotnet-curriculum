# Custom DTO: Available Materials

In this chapter, we'll implement a feature to get all available materials (those that are not checked out and not removed from circulation). This will demonstrate how DTOs can help us create more complex responses that involve filtering on related data.

## Understanding the Challenge

A material is considered "available" if:
1. It is not removed from circulation (`OutOfCirculationSince` is null)
2. It is not currently checked out (all of its checkouts have a `ReturnDate`)

The second condition is more complex because we need to check that *all* of a material's checkouts have a `ReturnDate`. This is where LINQ's `All` method becomes useful.

## Enhancing the MaterialDto

First, let's enhance our `MaterialDto` to include an `IsAvailable` property. Open the `DTOs/MaterialDto.cs` file and add the following property:

```csharp
public class MaterialDto
{
    public int Id { get; set; }
    public string MaterialName { get; set; }
    public int MaterialTypeId { get; set; }
    public MaterialTypeDto MaterialType { get; set; }
    public int GenreId { get; set; }
    public GenreDto Genre { get; set; }
    public DateTime? OutOfCirculationSince { get; set; }
    public List<CheckoutDto> Checkouts { get; set; }

    // Added property
    public bool IsAvailable { get; set; }
}
```

## Implementing the Endpoint

Now, let's implement the endpoint to get all available materials. Open the `Endpoints/MaterialEndpoints.cs` file and add the following method:

```csharp
// Add this to the MapMaterialEndpoints method
app.MapGet("/materials/available", GetAvailableMaterials);

// Add this method to the MaterialEndpoints class
private static IResult GetAvailableMaterials(LoncotesLibraryDbContext db, IMapper mapper)
{
    var availableMaterials = db.Materials
        .Include(m => m.Genre)
        .Include(m => m.MaterialType)
        .Include(m => m.Checkouts)
        .Where(m => m.OutOfCirculationSince == null)
        .Where(m => m.Checkouts.All(co => co.ReturnDate != null))
        .ToList();

    var materialDtos = mapper.Map<List<MaterialDto>>(availableMaterials);

    // Set IsAvailable to true for all materials in this list
    foreach (var material in materialDtos)
    {
        material.IsAvailable = true;
    }

    return Results.Ok(materialDtos);
}
```

Let's break down what's happening here:

1. We start by getting all materials that are in circulation and have all checkouts returned.
2. We use AutoMapper to map the materials to `MaterialDto` objects.
3. We set the `IsAvailable` property to true for all materials in the list.
4. We return the list of available materials.

The key part is the second `Where` clause:

```csharp
.Where(m => m.Checkouts.All(co => co.ReturnDate != null))
```

This uses LINQ's `All` method to check that all of a material's checkouts have a `ReturnDate`. If a material has no checkouts, `All` returns true (vacuously), which is what we want - a material with no checkouts is available.

## Testing the Endpoint

To test this endpoint, you'll need to have some materials in your database, some of which are checked out and some of which are not. You can use the endpoints we created in the previous chapter to add materials and manage checkouts.

Try calling the `/materials/available` endpoint and observe the response. You should see a list of available materials with their `IsAvailable` property set to true.

## Adding a Due Date Calculation

Let's enhance our available materials feature by adding a calculated property to show when the material would be due if checked out today. This will help patrons plan their checkouts.

First, let's create a new DTO specifically for available materials:

```csharp
// DTOs/AvailableMaterialDto.cs
public class AvailableMaterialDto
{
    public int Id { get; set; }
    public string MaterialName { get; set; }
    public int MaterialTypeId { get; set; }
    public MaterialTypeDto MaterialType { get; set; }
    public int GenreId { get; set; }
    public GenreDto Genre { get; set; }

    // Calculated property to show when the material will be due if checked out today
    public DateTime DueDate => DateTime.Today.AddDays(MaterialType.CheckoutDays);
}
```

Now, let's update our endpoint to use this new DTO:

```csharp
// Update the GetAvailableMaterials method in MaterialEndpoints.cs
private static IResult GetAvailableMaterials(LoncotesLibraryDbContext db, IMapper mapper)
{
    var availableMaterials = db.Materials
        .Include(m => m.Genre)
        .Include(m => m.MaterialType)
        .Include(m => m.Checkouts)
        .Where(m => m.OutOfCirculationSince == null)
        .Where(m => m.Checkouts.All(co => co.ReturnDate != null))
        .ToList();

    // Map to the new AvailableMaterialDto
    var availableMaterialDtos = mapper.Map<List<AvailableMaterialDto>>(availableMaterials);

    return Results.Ok(availableMaterialDtos);
}
```

We also need to update our AutoMapper profile to include the mapping for the new DTO:

```csharp
// Mapping/AutoMapperProfiles.cs
public class AutoMapperProfiles : Profile
{
    public AutoMapperProfiles()
    {
        CreateMap<Material, MaterialDto>();
        CreateMap<MaterialType, MaterialTypeDto>();
        CreateMap<Genre, GenreDto>();
        CreateMap<Patron, PatronDto>();
        CreateMap<Checkout, CheckoutDto>();

        // Add this mapping
        CreateMap<Material, AvailableMaterialDto>();
    }
}
```

Now, when a client calls the `/materials/available` endpoint, they'll get a list of available materials with a calculated `DueDate` property showing when each material would be due if checked out today.

## The Power of DTOs for Calculated Properties

This example demonstrates the power of DTOs for creating calculated properties. By using DTOs, we can:

1. Add properties that don't exist in our database models
2. Calculate values based on related data
3. Customize our responses for specific use cases

In this case, we're providing clients with useful information about material availability that isn't directly stored in our database.

## Summary

In this chapter, we've implemented a feature to get all available materials using DTOs with calculated properties. We've seen how DTOs can help us create more complex responses that involve filtering on related data and calculating values based on that data.

In the next chapter, we'll explore how to use DTOs to handle more complex scenarios, such as overdue checkouts.

Up Next: [Custom DTO: Overdue Checkouts](./loncotes-dto-overdue-checkouts.md)