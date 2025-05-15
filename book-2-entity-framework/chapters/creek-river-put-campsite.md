# Chapter to PUT a campsite

In this chapter, we'll create an API endpoint to update an existing campsite in our database using Entity Framework Core. This will allow clients to modify campsite information by sending HTTP PUT requests to our API.

## Understanding HTTP PUT Requests

HTTP PUT is one of the HTTP methods defined in the HTTP protocol. It is used to update a resource on the server by replacing the entire resource with a new representation. In RESTful APIs, PUT requests are typically used to update existing resources.

When a client sends a PUT request to our API, it includes a JSON payload in the request body that contains the updated data for the resource. Our API will process this data, update the resource in the database, and return an appropriate response.

## Creating the PUT Endpoint

Let's create an endpoint to update a campsite in our database:

1. Open the `Program.cs` file.

2. Add the following endpoint after the existing endpoints:

```csharp
app.MapPut("/api/campsites/{id}", (CreekRiverDbContext db, int id, Campsite campsite) =>
{
    Campsite campsiteToUpdate = db.Campsites.SingleOrDefault(c => c.Id == id);
    if (campsiteToUpdate == null)
    {
        return Results.NotFound();
    }
    campsiteToUpdate.Nickname = campsite.Nickname;
    campsiteToUpdate.CampsiteTypeId = campsite.CampsiteTypeId;
    campsiteToUpdate.ImageUrl = campsite.ImageUrl;

    db.SaveChanges();
    return Results.NoContent();
});
```

Let's break down this code:

- `app.MapPut("/api/campsites/{id}", ...)`: This maps HTTP PUT requests to the `/api/campsites/{id}` URL to our handler function.

- `(CreekRiverDbContext db, int id, Campsite campsite) => { ... }`: This is the handler function. The `CreekRiverDbContext` parameter is injected by ASP.NET Core's dependency injection system, the `id` parameter is bound to the `{id}` parameter in the URL, and the `Campsite` parameter is bound to the JSON payload in the request body.

- `Campsite campsiteToUpdate = db.Campsites.SingleOrDefault(c => c.Id == id)`: This retrieves the campsite with the specified ID from the database. `SingleOrDefault` returns `null` if no matching campsite is found.

- `if (campsiteToUpdate == null) { return Results.NotFound(); }`: This checks if the campsite was found. If not, it returns a 404 Not Found response.

- `campsiteToUpdate.Nickname = campsite.Nickname; ...`: This updates the properties of the existing campsite with the values from the incoming campsite.

- `db.SaveChanges()`: This saves the changes to the database. It executes an SQL UPDATE statement to update the campsite in the `Campsites` table.

- `return Results.NoContent()`: This returns a 204 No Content response, indicating that the request was successful but there is no content to return.

## Understanding Entity Framework Core's Update Operation

When you modify the properties of an entity that is being tracked by the DbContext, Entity Framework Core automatically detects these changes. When you call `SaveChanges`, EF Core executes an UPDATE statement to apply these changes to the database.

This is different from the `Add` and `Remove` operations, where you explicitly tell EF Core to start tracking an entity for addition or deletion. For updates, you simply modify the properties of an entity that is already being tracked.

## Testing the Endpoint

Now that we've created our endpoint, let's test it:

1. Run the application with `dotnet run` or by pressing F5 in Visual Studio.

2. Use a tool like Yaak to send a PUT request to `https://localhost:<port>/api/campsites/1` (replace `1` with the ID of a campsite you want to update) with a JSON payload like:

```json
{
  "id": 1,
  "nickname": "Updated Nickname",
  "imageUrl": "https://example.com/updated-image.jpg",
  "campsiteTypeId": 2
}
```

3. You should receive a 204 No Content response if the campsite was successfully updated, or a 404 Not Found response if no campsite with the specified ID was found.

## Adding Validation

Our current endpoint accepts any campsite data without validation. Let's add some basic validation to ensure that the required fields are provided and that the campsite type exists:

```csharp
app.MapPut("/api/campsites/{id}", (CreekRiverDbContext db, int id, Campsite campsite) =>
{
    // Validate required fields
    if (string.IsNullOrEmpty(campsite.Nickname))
    {
        return Results.BadRequest("Nickname is required.");
    }

    if (campsite.CampsiteTypeId <= 0)
    {
        return Results.BadRequest("Valid CampsiteTypeId is required.");
    }

    // Check if the CampsiteTypeId exists
    var campsiteType = db.CampsiteTypes.Find(campsite.CampsiteTypeId);
    if (campsiteType == null)
    {
        return Results.BadRequest($"CampsiteType with ID {campsite.CampsiteTypeId} does not exist.");
    }

    Campsite campsiteToUpdate = db.Campsites.SingleOrDefault(c => c.Id == id);
    if (campsiteToUpdate == null)
    {
        return Results.NotFound();
    }

    campsiteToUpdate.Nickname = campsite.Nickname;
    campsiteToUpdate.CampsiteTypeId = campsite.CampsiteTypeId;
    campsiteToUpdate.ImageUrl = campsite.ImageUrl;

    db.SaveChanges();
    return Results.NoContent();
});
```

This code adds validation to ensure that:
1. The `Nickname` field is not null or empty.
2. The `CampsiteTypeId` is a positive number.
3. The `CampsiteTypeId` references an existing campsite type.

## Handling Exceptions

Our current endpoint doesn't handle exceptions that might occur during the database operation. Let's add exception handling:

```csharp
app.MapPut("/api/campsites/{id}", (CreekRiverDbContext db, int id, Campsite campsite) =>
{
    // Validation code...

    try
    {
        Campsite campsiteToUpdate = db.Campsites.SingleOrDefault(c => c.Id == id);
        if (campsiteToUpdate == null)
        {
            return Results.NotFound();
        }

        campsiteToUpdate.Nickname = campsite.Nickname;
        campsiteToUpdate.CampsiteTypeId = campsite.CampsiteTypeId;
        campsiteToUpdate.ImageUrl = campsite.ImageUrl;

        db.SaveChanges();
        return Results.NoContent();
    }
    catch (DbUpdateException ex)
    {
        return Results.BadRequest($"Error updating campsite: {ex.Message}");
    }
});
```

This code catches `DbUpdateException`, which is thrown when there's an error saving changes to the database, such as a constraint violation.

## Using DTOs for Input and Output

In a real-world application, you might want to use DTOs (Data Transfer Objects) for input and output to decouple your API contract from your database schema. Here's how you might modify the endpoint to use DTOs:

```csharp
app.MapPut("/api/campsites/{id}", (CreekRiverDbContext db, int id, CampsiteUpdateDTO campsiteDTO) =>
{
    // Validate DTO...

    try
    {
        Campsite campsiteToUpdate = db.Campsites.SingleOrDefault(c => c.Id == id);
        if (campsiteToUpdate == null)
        {
            return Results.NotFound();
        }

        // Map DTO to entity
        campsiteToUpdate.Nickname = campsiteDTO.Nickname;
        campsiteToUpdate.CampsiteTypeId = campsiteDTO.CampsiteTypeId;
        campsiteToUpdate.ImageUrl = campsiteDTO.ImageUrl;

        db.SaveChanges();
        return Results.NoContent();
    }
    catch (DbUpdateException ex)
    {
        return Results.BadRequest($"Error updating campsite: {ex.Message}");
    }
});
```

You would need to define a `CampsiteUpdateDTO` class that represents the data needed to update a campsite:

```csharp
public class CampsiteUpdateDTO
{
    public string Nickname { get; set; }
    public string ImageUrl { get; set; }
    public int CampsiteTypeId { get; set; }
}
```

## Checking for Existing Reservations

Before updating a campsite, you might want to check if there are any active reservations for that campsite. If there are, you might want to prevent certain updates or handle them differently:

```csharp
app.MapPut("/api/campsites/{id}", (CreekRiverDbContext db, int id, Campsite campsite) =>
{
    // Validation code...

    try
    {
        Campsite campsiteToUpdate = db.Campsites
            .Include(c => c.Reservations)
            .SingleOrDefault(c => c.Id == id);

        if (campsiteToUpdate == null)
        {
            return Results.NotFound();
        }

        // Check for active reservations
        var activeReservations = campsiteToUpdate.Reservations
            .Where(r => r.CheckoutDate > DateTime.Now)
            .ToList();

        if (activeReservations.Any() && campsiteToUpdate.CampsiteTypeId != campsite.CampsiteTypeId)
        {
            return Results.BadRequest("Cannot change campsite type while there are active reservations.");
        }

        campsiteToUpdate.Nickname = campsite.Nickname;
        campsiteToUpdate.CampsiteTypeId = campsite.CampsiteTypeId;
        campsiteToUpdate.ImageUrl = campsite.ImageUrl;

        db.SaveChanges();
        return Results.NoContent();
    }
    catch (DbUpdateException ex)
    {
        return Results.BadRequest($"Error updating campsite: {ex.Message}");
    }
});
```

This code includes the reservations when retrieving the campsite, then checks if there are any active reservations (reservations with a checkout date in the future). If there are, and the campsite type is being changed, it returns a 400 Bad Request response with an error message.

## The Complete Endpoint

Here's the complete endpoint with validation, exception handling, and checking for active reservations:

```csharp
app.MapPut("/api/campsites/{id}", (CreekRiverDbContext db, int id, Campsite campsite) =>
{
    // Validate required fields
    if (string.IsNullOrEmpty(campsite.Nickname))
    {
        return Results.BadRequest("Nickname is required.");
    }

    if (campsite.CampsiteTypeId <= 0)
    {
        return Results.BadRequest("Valid CampsiteTypeId is required.");
    }

    // Check if the CampsiteTypeId exists
    var campsiteType = db.CampsiteTypes.Find(campsite.CampsiteTypeId);
    if (campsiteType == null)
    {
        return Results.BadRequest($"CampsiteType with ID {campsite.CampsiteTypeId} does not exist.");
    }

    try
    {
        Campsite campsiteToUpdate = db.Campsites
            .Include(c => c.Reservations)
            .SingleOrDefault(c => c.Id == id);

        if (campsiteToUpdate == null)
        {
            return Results.NotFound();
        }

        // Check for active reservations
        var activeReservations = campsiteToUpdate.Reservations
            .Where(r => r.CheckoutDate > DateTime.Now)
            .ToList();

        if (activeReservations.Any() && campsiteToUpdate.CampsiteTypeId != campsite.CampsiteTypeId)
        {
            return Results.BadRequest("Cannot change campsite type while there are active reservations.");
        }

        campsiteToUpdate.Nickname = campsite.Nickname;
        campsiteToUpdate.CampsiteTypeId = campsite.CampsiteTypeId;
        campsiteToUpdate.ImageUrl = campsite.ImageUrl;

        db.SaveChanges();
        return Results.NoContent();
    }
    catch (DbUpdateException ex)
    {
        return Results.BadRequest($"Error updating campsite: {ex.Message}");
    }
});
```

## Conclusion

In this chapter, we've created an API endpoint to update a campsite in our database using Entity Framework Core. We've learned how to:

1. Create a PUT endpoint to update a resource
2. Use Entity Framework Core to update an entity in the database
3. Return appropriate HTTP responses based on the result of the operation
4. Add validation to ensure that the required fields are provided and that the campsite type exists
5. Handle exceptions that might occur during the database operation
6. Check for related entities before updating an entity
7. Use DTOs for input and output to decouple the API contract from the database schema

These concepts are fundamental to creating RESTful APIs with ASP.NET Core and Entity Framework Core.

In the next chapter, we'll create an endpoint to retrieve reservations with their related data.

Up Next: [Chapter to GET reservations which introduces LINQ methods ThenInclude](./creek-river-get-reservations.md)

## 🔍 Additional Materials

- [ASP.NET Core Minimal APIs](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis)
- [Entity Framework Core - Saving Data](https://docs.microsoft.com/en-us/ef/core/saving/)
- [Entity Framework Core - Change Tracking](https://docs.microsoft.com/en-us/ef/core/change-tracking/)
- [HTTP PUT Method](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/PUT)
- [RESTful API Design](https://docs.microsoft.com/en-us/azure/architecture/best-practices/api-design)