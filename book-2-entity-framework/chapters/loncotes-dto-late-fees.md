# Custom DTO: Late Fees

In this chapter, we'll implement a feature to calculate late fees for overdue materials. This will demonstrate how DTOs can encapsulate complex business logic and provide calculated properties that don't exist in the database.

## Understanding Late Fees

The Loncotes County Library charges a late fee of $0.50 per day for materials that are returned after their due date. To calculate the late fee, we need to:

1. Calculate the due date by adding the material type's checkout days to the checkout date
2. Calculate the number of days between the return date and the due date
3. Multiply the number of days by the late fee rate

## Creating a DTO with Late Fee Calculation

Let's create a DTO specifically for checkouts that includes a calculated property for late fees:

```csharp
// DTOs/CheckoutWithLateFeeDto.cs
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
    public bool Paid { get; set; }

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
        CreateMap<Checkout, OverdueCheckoutDto>();

        // Add this mapping
        CreateMap<Checkout, CheckoutWithLateFeeDto>();
    }
}
```

## Implementing the Endpoint

Now, let's implement an endpoint to get all checkouts with late fees. Open the `Endpoints/CheckoutEndpoints.cs` file and add the following method:

```csharp
// Add this to the MapCheckoutEndpoints method
app.MapGet("/checkouts/with-late-fees", GetCheckoutsWithLateFees);

// Add this method to the CheckoutEndpoints class
private static IResult GetCheckoutsWithLateFees(LoncotesLibraryDbContext db, IMapper mapper)
{
    var checkouts = db.Checkouts
        .Include(c => c.Material)
            .ThenInclude(m => m.MaterialType)
        .Include(c => c.Patron)
        .ToList();

    var checkoutDtos = mapper.Map<List<CheckoutWithLateFeeDto>>(checkouts);

    // Filter to only include checkouts with late fees
    var checkoutsWithLateFees = checkoutDtos.Where(c => c.LateFee.HasValue).ToList();

    return Results.Ok(checkoutsWithLateFees);
}
```

Let's break down what's happening here:

1. We start by getting all checkouts and including their related data (Material, MaterialType, and Patron).
2. We use AutoMapper to map the checkouts to `CheckoutWithLateFeeDto` objects.
3. We filter the results to only include checkouts with late fees.
4. We return the filtered list of checkouts with late fees.

## Implementing an Endpoint to Pay a Late Fee

Now, let's implement an endpoint to mark a late fee as paid. Open the `Endpoints/CheckoutEndpoints.cs` file and add the following method:

```csharp
// Add this to the MapCheckoutEndpoints method
app.MapPut("/checkouts/{id}/pay", PayLateFee);

// Add this method to the CheckoutEndpoints class
private static IResult PayLateFee(LoncotesLibraryDbContext db, int id)
{
    var checkout = db.Checkouts.Find(id);

    if (checkout == null)
    {
        return Results.NotFound();
    }

    checkout.Paid = true;
    db.SaveChanges();

    return Results.NoContent();
}
```

## Calculating a Patron's Total Balance

Let's create a DTO that includes a calculated property for a patron's total unpaid late fees:

```csharp
// DTOs/PatronWithBalanceDto.cs
public class PatronWithBalanceDto
{
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
        CreateMap<Checkout, OverdueCheckoutDto>();
        CreateMap<Checkout, CheckoutWithLateFeeDto>();

        // Add this mapping
        CreateMap<Patron, PatronWithBalanceDto>();
    }
}
```

Now, let's implement an endpoint to get a patron with their balance. Open the `Endpoints/PatronEndpoints.cs` file and add the following method:

```csharp
// Add this to the MapPatronEndpoints method
app.MapGet("/patrons/{id}/balance", GetPatronBalance);

// Add this method to the PatronEndpoints class
private static IResult GetPatronBalance(LoncotesLibraryDbContext db, IMapper mapper, int id)
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

    return Results.Ok(mapper.Map<PatronWithBalanceDto>(patron));
}
```

## Testing the Endpoints

To test these endpoints, you'll need to have some checkouts in your database with various return dates. You can use the endpoints we created in the previous chapters to add and manage checkouts.

Try calling the `/checkouts/with-late-fees` endpoint and observe the response. You should see a list of checkouts with their late fees.

Then, try calling the `/patrons/{id}/balance` endpoint for a patron with overdue checkouts, and observe their total balance.

## The Power of DTOs for Complex Business Logic

This example demonstrates how DTOs can encapsulate complex business logic and provide calculated properties based on that logic. By using DTOs, we can:

1. Calculate values based on business rules
2. Handle different use cases with different DTOs
3. Provide aggregated information that spans multiple entities

In this case, we're providing librarians with useful information about late fees and patron balances that isn't directly stored in our database.

## Summary

In this chapter, we've implemented features to calculate late fees for overdue materials and to get a patron's total balance. We've seen how DTOs can help us encapsulate complex business logic and provide calculated properties based on that logic.

In the next chapter, we'll explore how to use AutoMapper to simplify the mapping between our models and DTOs even further.

Up Next: [Quieter Code with AutoMapper](./loncotes-dto-automapper.md)