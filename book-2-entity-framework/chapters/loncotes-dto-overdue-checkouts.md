# Overdue Checkouts with DTOs

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

## Implementing the Endpoint

Now, let's implement an endpoint to get all overdue checkouts:

```csharp
app.MapGet("/api/checkouts/overdue", (LoncotesLibraryDbContext db) =>
{
    return db.Checkouts
        .Include(c => c.Material)
            .ThenInclude(m => m.MaterialType)
        .Include(c => c.Patron)
        .Where(c => c.ReturnDate == null)
        .Where(c => (DateTime.Today - c.CheckoutDate).Days > c.Material.MaterialType.CheckoutDays)
        .Select(c => new OverdueCheckoutDto
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
                // FullName is automatically calculated
            },
            CheckoutDate = c.CheckoutDate
        })
        .ToList();
});
```

Let's break down what's happening here:

1. We start by getting all checkouts and including their related data (Material, MaterialType, and Patron).
2. We filter out checkouts that have been returned.
3. We then filter for checkouts where the number of days since checkout exceeds the allowed checkout days.
4. Finally, we transform the results into `OverdueCheckoutDto` objects.

The key part is the second `Where` clause:

```csharp
.Where(c => (DateTime.Today - c.CheckoutDate).Days > c.Material.MaterialType.CheckoutDays)
```

This calculates the number of days since checkout and compares it to the allowed checkout days for the material type.

## Adding Email Notifications for Overdue Checkouts

Let's enhance our DTO to include a template for email notifications:

```csharp
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

    // Calculated property for email subject
    public string EmailSubject => $"Overdue Material: {Material.MaterialName}";

    // Calculated property for email body
    public string EmailBody => $"Dear {Patron.FirstName} {Patron.LastName},\n\n" +
                              $"Our records indicate that you have an overdue item: {Material.MaterialName}.\n" +
                              $"This item was due on {DueDate.ToShortDateString()} and is now {DaysOverdue} days overdue.\n\n" +
                              "Please return this item to the library as soon as possible.\n\n" +
                              "Thank you,\n" +
                              "Loncotes County Library";
}
```

We've added two more calculated properties:
- `EmailSubject`: A subject line for an email notification
- `EmailBody`: A body for an email notification that includes the patron's name, the material name, the due date, and the number of days overdue

Now, when a librarian retrieves the list of overdue checkouts, they'll also get templates for email notifications that they can send to patrons.

## Implementing a Bulk Email Endpoint

Let's take this a step further and implement an endpoint to send email notifications for all overdue checkouts:

```csharp
app.MapPost("/api/checkouts/overdue/send-notifications", (LoncotesLibraryDbContext db) =>
{
    var overdueCheckouts = db.Checkouts
        .Include(c => c.Material)
            .ThenInclude(m => m.MaterialType)
        .Include(c => c.Patron)
        .Where(c => c.ReturnDate == null)
        .Where(c => (DateTime.Today - c.CheckoutDate).Days > c.Material.MaterialType.CheckoutDays)
        .Select(c => new OverdueCheckoutDto
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
            CheckoutDate = c.CheckoutDate
        })
        .ToList();

    // In a real application, you would send emails here
    // For this example, we'll just return the number of notifications that would be sent

    return Results.Ok(new { NotificationsSent = overdueCheckouts.Count });
});
```

This endpoint retrieves all overdue checkouts and would send email notifications to the patrons. In a real application, you would integrate with an email service to send the actual emails.

## The Power of DTOs for Business Logic

This example demonstrates how DTOs can encapsulate business logic and provide calculated properties based on that logic. By using DTOs, we can:

1. Calculate complex values based on multiple related entities
2. Provide formatted output for specific use cases
3. Encapsulate business rules in a reusable way

In this case, we're providing librarians with useful information about overdue checkouts and tools to notify patrons.

## Testing the Endpoint

To test this endpoint, you'll need to have some checkouts in your database that are overdue. You can use the endpoints we created in the previous chapters to add checkouts, and then wait until they become overdue (or manipulate the checkout dates in the database).

Try calling the `/api/checkouts/overdue` endpoint and observe the response. You should see a list of overdue checkouts with their due dates, days overdue, and email templates.

In the next chapter, we'll explore how to use DTOs to calculate late fees for overdue materials.

Up Next: [Late Fees with DTOs](./loncotes-dto-late-fees.md)