# Late Fees with DTOs

In this chapter, we'll implement a feature to calculate late fees for overdue materials. This will demonstrate how DTOs can encapsulate complex business logic and provide calculated properties that don't exist in the database.

## Understanding Late Fees

The Loncotes County Library charges a late fee of $0.50 per day for materials that are returned after their due date. To calculate the late fee, we need to:

1. Calculate the due date by adding the material type's checkout days to the checkout date
2. Calculate the number of days between the return date and the due date
3. Multiply the number of days by the late fee rate

## Creating a DTO with Late Fee Calculation

Let's create a DTO specifically for checkouts that includes a calculated property for late fees:

```csharp
public class CheckoutWithLateFeeDto
{
    private static decimal _lateFeePerDay = 0.50M; // $0.50 per day

    public int Id { get; set; }
    public int MaterialId { get; set; }
    public MaterialDto Material { get; set; }
    public int PatronId { get; set; }
    public PatronDto Patron { get; set; }
    public DateTime CheckoutDate { get; set; }
    public DateTime? ReturnDate { get; set; }

    // Calculated property for the due date
    public DateTime DueDate => CheckoutDate.AddDays(Material.MaterialType.CheckoutDays);

    // Calculated property for late fee
    public decimal? LateFee
    {
        get
        {
            // If the material hasn't been returned yet, use today's date for calculation
            DateTime actualReturnDate = ReturnDate ?? DateTime.Today;

            // Calculate days late
            int daysLate = (actualReturnDate - DueDate).Days;

            // If not late, no fee
            if (daysLate <= 0)
            {
                return null;
            }

            // Calculate fee
            return daysLate * _lateFeePerDay;
        }
    }
}
```

Notice how we've added:
- A private static field `_lateFeePerDay` to store the late fee rate
- A calculated property `DueDate` to determine when the material was due
- A calculated property `LateFee` that computes the late fee based on how many days the material is/was late

The `LateFee` property handles several cases:
- If the material has been returned, it uses the actual return date
- If the material hasn't been returned, it uses today's date to calculate the current accrued fee
- If the material was returned on time, it returns null (no fee)

## Implementing the Endpoint

Now, let's implement an endpoint to get all checkouts with late fees:

```csharp
app.MapGet("/api/checkouts", (LoncotesLibraryDbContext db) =>
{
    return db.Checkouts
        .Include(c => c.Material)
            .ThenInclude(m => m.MaterialType)
        .Include(c => c.Patron)
        .Select(c => new CheckoutWithLateFeeDto
        {
            Id = c.Id,
            MaterialId = c.MaterialId,
            Material = new MaterialDto
            {
                Id = c.Material.Id,
                MaterialName = c.Material.MaterialName,
                MaterialTypeId = c.Material.MaterialTypeId,
                MaterialType = new MaterialTypeDto
                {
                    Id = c.Material.MaterialType.Id,
                    Name = c.Material.MaterialType.Name,
                    CheckoutDays = c.Material.MaterialType.CheckoutDays
                },
                GenreId = c.Material.GenreId,
                OutOfCirculationSince = c.Material.OutOfCirculationSince
            },
            PatronId = c.PatronId,
            Patron = new PatronDto
            {
                Id = c.Patron.Id,
                FirstName = c.Patron.FirstName,
                LastName = c.Patron.LastName,
                Address = c.Patron.Address,
                Email = c.Patron.Email,
                IsActive = c.Patron.IsActive
            },
            CheckoutDate = c.CheckoutDate,
            ReturnDate = c.ReturnDate
        })
        .ToList();
});
```

This endpoint retrieves all checkouts and includes the calculated late fee for each one.

## Handling Potential Regression Issues

When we add calculated properties to DTOs that depend on related entities, we need to be careful about how we use those DTOs in other parts of our application. For example, if we have an endpoint that returns a material with its checkouts, we need to ensure that the `Material` and `MaterialType` properties are set for each checkout, or we'll get a null reference exception when the `LateFee` property tries to access them.

One way to handle this is to create separate DTOs for different use cases. Let's create a simplified checkout DTO for use in the material details endpoint:

```csharp
public class SimpleCheckoutDto
{
    public int Id { get; set; }
    public int PatronId { get; set; }
    public PatronDto Patron { get; set; }
    public DateTime CheckoutDate { get; set; }
    public DateTime? ReturnDate { get; set; }
}
```

Now, let's update the material details endpoint to use this simplified DTO:

```csharp
app.MapGet("/api/materials/{id}", (LoncotesLibraryDbContext db, int id) =>
{
    var material = db.Materials
        .Include(m => m.Genre)
        .Include(m => m.MaterialType)
        .Include(m => m.Checkouts)
            .ThenInclude(c => c.Patron)
        .FirstOrDefault(m => m.Id == id);

    if (material == null)
    {
        return Results.NotFound();
    }

    return Results.Ok(new MaterialDto
    {
        Id = material.Id,
        MaterialName = material.MaterialName,
        MaterialTypeId = material.MaterialTypeId,
        MaterialType = new MaterialTypeDto
        {
            Id = material.MaterialType.Id,
            Name = material.MaterialType.Name,
            CheckoutDays = material.MaterialType.CheckoutDays
        },
        GenreId = material.GenreId,
        Genre = new GenreDto
        {
            Id = material.Genre.Id,
            Name = material.Genre.Name
        },
        OutOfCirculationSince = material.OutOfCirculationSince,
        Checkouts = material.Checkouts.Select(c => new SimpleCheckoutDto
        {
            Id = c.Id,
            PatronId = c.PatronId,
            Patron = new PatronDto
            {
                Id = c.Patron.Id,
                FirstName = c.Patron.FirstName,
                LastName = c.Patron.LastName,
                Address = c.Patron.Address,
                Email = c.Patron.Email,
                IsActive = c.Patron.IsActive
            },
            CheckoutDate = c.CheckoutDate,
            ReturnDate = c.ReturnDate
        }).ToList()
    });
});
```

By using a different DTO for checkouts in this context, we avoid the potential null reference exception.

## Adding a Paid Flag to Checkouts

Let's enhance our model and DTOs to track whether a late fee has been paid:

```csharp
// Add to the Checkout model
public bool Paid { get; set; }

// Add to the CheckoutWithLateFeeDto
public bool Paid { get; set; }
```

Now, let's implement an endpoint to mark a late fee as paid:

```csharp
app.MapPut("/api/checkouts/{id}/pay", (LoncotesLibraryDbContext db, int id) =>
{
    var checkout = db.Checkouts.Find(id);

    if (checkout == null)
    {
        return Results.NotFound();
    }

    checkout.Paid = true;
    db.SaveChanges();

    return Results.NoContent();
});
```

## Calculating a Patron's Total Balance

Let's create a DTO that includes a calculated property for a patron's total unpaid late fees:

```csharp
public class PatronWithBalanceDto
{
    private static decimal _lateFeePerDay = 0.50M; // $0.50 per day

    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Address { get; set; }
    public string Email { get; set; }
    public bool IsActive { get; set; }

    // Calculated property for full name
    public string FullName => $"{FirstName} {LastName}";

    public List<CheckoutWithLateFeeDto> Checkouts { get; set; }

    // Calculated property for total balance
    public decimal Balance
    {
        get
        {
            if (Checkouts == null)
            {
                return 0;
            }

            return Checkouts
                .Where(c => c.LateFee.HasValue && !c.Paid)
                .Sum(c => c.LateFee.Value);
        }
    }
}
```

Now, let's implement an endpoint to get a patron with their balance:

```csharp
app.MapGet("/api/patrons/{id}/balance", (LoncotesLibraryDbContext db, int id) =>
{
    var patron = db.Patrons
        .Include(p => p.Checkouts)
            .ThenInclude(c => c.Material)
                .ThenInclude(m => m.MaterialType)
        .FirstOrDefault(p => p.Id == id);

    if (patron == null)
    {
        return Results.NotFound();
    }

    return Results.Ok(new PatronWithBalanceDto
    {
        Id = patron.Id,
        FirstName = patron.FirstName,
        LastName = patron.LastName,
        Address = patron.Address,
        Email = patron.Email,
        IsActive = patron.IsActive,
        Checkouts = patron.Checkouts.Select(c => new CheckoutWithLateFeeDto
        {
            Id = c.Id,
            MaterialId = c.MaterialId,
            Material = new MaterialDto
            {
                Id = c.Material.Id,
                MaterialName = c.Material.MaterialName,
                MaterialTypeId = c.Material.MaterialTypeId,
                MaterialType = new MaterialTypeDto
                {
                    Id = c.Material.MaterialType.Id,
                    Name = c.Material.MaterialType.Name,
                    CheckoutDays = c.Material.MaterialType.CheckoutDays
                },
                GenreId = c.Material.GenreId,
                OutOfCirculationSince = c.Material.OutOfCirculationSince
            },
            PatronId = c.PatronId,
            CheckoutDate = c.CheckoutDate,
            ReturnDate = c.ReturnDate,
            Paid = c.Paid
        }).ToList()
    });
});
```

This endpoint retrieves a patron with all their checkouts and calculates their total balance of unpaid late fees.

## The Power of DTOs for Complex Business Logic

This example demonstrates how DTOs can encapsulate complex business logic and provide calculated properties based on that logic. By using DTOs, we can:

1. Calculate values based on business rules
2. Handle different use cases with different DTOs
3. Provide aggregated information that spans multiple entities

In this case, we're providing librarians with useful information about late fees and patron balances that isn't directly stored in our database.

## Testing the Endpoints

To test these endpoints, you'll need to have some checkouts in your database with various return dates. You can use the endpoints we created in the previous chapters to add and manage checkouts.

Try calling the `/api/checkouts` endpoint and observe the response. You should see a list of checkouts with their late fees (if any). Then, try calling the `/api/patrons/{id}/balance` endpoint for a patron with overdue checkouts, and observe their total balance.

In the next chapter, we'll explore how to use AutoMapper to simplify the mapping between our models and DTOs.

Up Next: [Using AutoMapper with DTOs](./loncotes-dto-automapper.md)