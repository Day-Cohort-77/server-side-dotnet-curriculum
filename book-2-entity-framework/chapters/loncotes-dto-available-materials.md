# Available Materials with DTOs

In this chapter, we'll implement a feature to get all available materials (those that are not checked out and not removed from circulation). This will demonstrate how DTOs can help us create more complex responses that involve filtering on related data.

## Understanding the Challenge

A material is considered "available" if:
1. It is not removed from circulation (`OutOfCirculationSince` is null)
2. It is not currently checked out (all of its checkouts have a `ReturnDate`)

The second condition is more complex because we need to check that *all* of a material's checkouts have a `ReturnDate`. This is where LINQ's `All` method becomes useful.

## Implementing the Endpoint

Let's create an endpoint to get all available materials:

```csharp
app.MapGet("/materials/available", (LoncotesLibraryDbContext db) =>
{
    return db.Materials
        .Include(m => m.Genre)
        .Include(m => m.MaterialType)
        .Include(m => m.Checkouts)
        .Where(m => m.OutOfCirculationSince == null)
        .Where(m => m.Checkouts.All(co => co.ReturnDate != null))
        .Select(m => new MaterialDto
        {
            Id = m.Id,
            MaterialName = m.MaterialName,
            MaterialTypeId = m.MaterialTypeId,
            MaterialType = new MaterialTypeDto
            {
                Id = m.MaterialType.Id,
                Name = m.MaterialType.Name,
                CheckoutDays = m.MaterialType.CheckoutDays
            },
            GenreId = m.GenreId,
            Genre = new GenreDto
            {
                Id = m.Genre.Id,
                Name = m.Genre.Name
            },
            OutOfCirculationSince = m.OutOfCirculationSince
        })
        .ToList();
});
```

Let's break down what's happening here:

1. We start by getting all materials and including their related data (Genre, MaterialType, and Checkouts).
2. We filter out materials that are removed from circulation.
3. We then filter out materials that have any checkouts without a return date.
4. Finally, we transform the results into `MaterialDto` objects.

The key part is the second `Where` clause:

```csharp
.Where(m => m.Checkouts.All(co => co.ReturnDate != null))
```

This uses LINQ's `All` method to check that all of a material's checkouts have a `ReturnDate`. If a material has no checkouts, `All` returns true (vacuously), which is what we want - a material with no checkouts is available.

## Creating a Custom DTO for Available Materials

While the above solution works, we might want to include additional information about the material's availability in our response. Let's create a custom DTO for available materials:

```csharp
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

    // Calculated property to show how many days the material can be checked out
    public int CheckoutPeriod => MaterialType.CheckoutDays;
}
```

Notice how we've added two calculated properties:
- `DueDate`: Shows when the material would be due if checked out today
- `CheckoutPeriod`: Shows how many days the material can be checked out

Now, let's update our endpoint to use this new DTO:

```csharp
app.MapGet("/materials/available", (LoncotesLibraryDbContext db) =>
{
    return db.Materials
        .Include(m => m.Genre)
        .Include(m => m.MaterialType)
        .Include(m => m.Checkouts)
        .Where(m => m.OutOfCirculationSince == null)
        .Where(m => m.Checkouts.All(co => co.ReturnDate != null))
        .Select(m => new AvailableMaterialDto
        {
            Id = m.Id,
            MaterialName = m.MaterialName,
            MaterialTypeId = m.MaterialTypeId,
            MaterialType = new MaterialTypeDto
            {
                Id = m.MaterialType.Id,
                Name = m.MaterialType.Name,
                CheckoutDays = m.MaterialType.CheckoutDays
            },
            GenreId = m.GenreId,
            Genre = new GenreDto
            {
                Id = m.Genre.Id,
                Name = m.Genre.Name
            }
        })
        .ToList();
});
```

Now, when a client calls this endpoint, they'll get a list of available materials with additional information about when each material would be due if checked out today and how long it can be checked out for.

## The Power of DTOs for Calculated Properties

This example demonstrates the power of DTOs for creating calculated properties. By using DTOs, we can:

1. Add properties that don't exist in our database models
2. Calculate values based on related data
3. Customize our responses for specific use cases

In this case, we're providing clients with useful information about material availability that isn't directly stored in our database.

## Testing the Endpoint

To test this endpoint, you'll need to have some materials in your database that are not checked out and not removed from circulation. You can use the endpoints we created in the previous chapter to add materials and manage checkouts.

Try calling the `/materials/available` endpoint and observe the response. You should see a list of available materials with their due dates and checkout periods.

In the next chapter, we'll explore how to use DTOs to handle more complex scenarios, such as overdue checkouts and late fees.

Up Next: [Overdue Checkouts with DTOs](./loncotes-dto-overdue-checkouts.md)