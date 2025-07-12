# Custom DTO: Overdue Checkouts

In this chapter, we'll implement a feature to get all overdue checkouts. This will demonstrate how DTOs can help us create complex calculated properties based on business logic.

## Understanding Overdue Checkouts

A checkout is considered "overdue" if:
1. The material has not been returned (`ReturnDate` is null)
2. The number of days since checkout exceeds the allowed checkout days for that material type

To determine if a checkout is overdue, we need to:
1. Calculate the due date by adding the material type's checkout days to the checkout date
2. Compare the due date to today's date

## Creating a DTO with Calculated Properties

Let's create a DTO specifically for overdue checkouts that includes calculated properties for the due date and days overdue:

```csharp
// DTOs/OverdueCheckoutDto.cs
public class OverdueCheckoutDto
{
    public int Id { get; set; }
    public int MaterialId { get; set; }
    public MaterialDto Material { get; set; }
    public int PatronId { get; set; }
    public PatronDto Patron { get; set; }
    public DateTime CheckoutDate { get; set; }

    // Calculated property for the due date
    public DateTime DueDate => CheckoutDate.AddDays(Material.MaterialType.CheckoutDays);

    // Calculated property for days overdue
    public int DaysOverdue => (DateTime.Today - DueDate).Days;
}
```

Notice how we've added two calculated properties:
- `DueDate`: Calculates when the material was due by adding the material type's checkout days to the checkout date
- `DaysOverdue`: Calculates how many days the material is overdue by comparing the due date to today's date

## Updating the AutoMapper Profile

Now, let's update our AutoMapper profile to include the mapping for the new DTO:

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
        CreateMap<Material, AvailableMaterialDto>();

        // Add this mapping
        CreateMap<Checkout, OverdueCheckoutDto>();
    }
}
```

## Implementing the Endpoint

Now, let's implement the endpoint to get all overdue checkouts. Open the `Endpoints/CheckoutEndpoints.cs` file and add the following method:

```csharp
// Add this to the MapCheckoutEndpoints method
app.MapGet("/checkouts/overdue", GetOverdueCheckouts);

// Add this method to the CheckoutEndpoints class
private static IResult GetOverdueCheckouts(LoncotesLibraryDbContext db, IMapper mapper)
{
    var overdueCheckouts = db.Checkouts
        .Include(c => c.Material)
            .ThenInclude(m => m.MaterialType)
        .Include(c => c.Patron)
        .Where(c => c.ReturnDate == null)
        .Where(c => (DateTime.Today - c.CheckoutDate).Days > c.Material.MaterialType.CheckoutDays)
        .ToList();

    return Results.Ok(mapper.Map<List<OverdueCheckoutDto>>(overdueCheckouts));
}
```

Let's break down what's happening here:

1. We start by getting all checkouts that haven't been returned.
2. We filter for checkouts where the number of days since checkout exceeds the allowed checkout days for the material type.
3. We use AutoMapper to map the checkouts to `OverdueCheckoutDto` objects.
4. We return the list of overdue checkouts.

The key part is the second `Where` clause:

```csharp
.Where(c => (DateTime.Today - c.CheckoutDate).Days > c.Material.MaterialType.CheckoutDays)
```

This calculates the number of days since checkout and compares it to the allowed checkout days for the material type.

## Testing the Endpoint

To test this endpoint, you'll need to have some checkouts in your database that are overdue. You can use the endpoints we created in the previous chapters to add checkouts, and then wait until they become overdue (or manipulate the checkout dates in the database).

Try calling the `/checkouts/overdue` endpoint and observe the response. You should see a list of overdue checkouts with their due dates, and days overdue.

## The Power of DTOs for Business Logic

This example demonstrates how DTOs can encapsulate business logic and provide calculated properties based on that logic. By using DTOs, we can:

1. Calculate complex values based on multiple related entities
2. Provide formatted output for specific use cases
3. Encapsulate business rules in a reusable way

In this case, we're providing librarians with useful information about overdue checkouts and tools to notify patrons.

## Summary

In this chapter, we've implemented a feature to get all overdue checkouts using DTOs with calculated properties. We've seen how DTOs can help us encapsulate business logic and provide calculated properties based on that logic.

In the next chapter, we'll explore how to use DTOs to calculate late fees for overdue materials.

Up Next: [Custom DTO: Late Fees](./loncotes-dto-late-fees.md)