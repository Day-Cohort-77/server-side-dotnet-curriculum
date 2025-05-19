# Deleting a campsite

In this chapter, we'll create an API endpoint to delete a campsite from our database using Entity Framework Core. This will allow clients to remove campsites by sending HTTP DELETE requests to our API.

## Understanding HTTP DELETE Requests

HTTP DELETE is one of the HTTP methods defined in the HTTP protocol. It is used to request the removal of a resource from the server. In RESTful APIs, DELETE requests are typically used to delete existing resources.

When a client sends a DELETE request to our API, it specifies the ID of the resource to delete in the URL. Our API will process this request, delete the resource from the database, and return an appropriate response.

## Creating the DELETE Endpoint

Let's create an endpoint to delete a campsite from our database:

1. Open the `Program.cs` file.

2. Add the following endpoint after the existing endpoints:

```csharp
app.MapDelete("/api/campsites/{id}", (CreekRiverDbContext db, int id) =>
{
    Campsite campsite = db.Campsites.SingleOrDefault(campsite => campsite.Id == id);
    if (campsite == null)
    {
        return Results.NotFound();
    }
    db.Campsites.Remove(campsite);
    db.SaveChanges();
    return Results.NoContent();
});
```

Let's break down this code:

- `app.MapDelete("/api/campsites/{id}", ...)`: This maps HTTP DELETE requests to the `/api/campsites/{id}` URL to our handler function.

- `(CreekRiverDbContext db, int id) => { ... }`: This is the handler function. The `CreekRiverDbContext` parameter is injected by ASP.NET Core's dependency injection system, and the `id` parameter is bound to the `{id}` parameter in the URL.

- `Campsite campsite = db.Campsites.SingleOrDefault(campsite => campsite.Id == id)`: This retrieves the campsite with the specified ID from the database. `SingleOrDefault` returns `null` if no matching campsite is found.

- `if (campsite == null) { return Results.NotFound(); }`: This checks if the campsite was found. If not, it returns a 404 Not Found response.

- `db.Campsites.Remove(campsite)`: This marks the campsite for deletion in the DbContext. At this point, the campsite is only marked for deletion but not yet removed from the database.

- `db.SaveChanges()`: This saves the changes to the database. It executes an SQL DELETE statement to remove the campsite from the `Campsites` table.

- `return Results.NoContent()`: This returns a 204 No Content response, indicating that the request was successful but there is no content to return.

## Understanding Entity Framework Core's Delete Operation

When you call `Remove` on a DbSet, Entity Framework Core marks the entity for deletion. When you call `SaveChanges`, EF Core executes a DELETE statement to remove the entity from the database.

If the entity has relationships with other entities, EF Core will handle those relationships according to the cascade delete behavior configured in your model. By default, EF Core will cascade delete related entities if the relationship is required (i.e., if the foreign key property is not nullable).

## Testing the Endpoint

Now that we've created our endpoint, let's test it:

1. Run the application with `dotnet run` or by pressing F5 in Visual Studio.

2. Use a tool like Yaak to send a DELETE request to `https://localhost:<port>/api/campsites/1` (replace `1` with the ID of a campsite you want to delete).

3. You should receive a 204 No Content response if the campsite was successfully deleted, or a 404 Not Found response if no campsite with the specified ID was found.

## Adding Validation and Error Handling

Our current endpoint doesn't handle exceptions that might occur during the database operation. Let's add exception handling:

```csharp
app.MapDelete("/api/campsites/{id}", (CreekRiverDbContext db, int id) =>
{
    try
    {
        Campsite campsite = db.Campsites.SingleOrDefault(campsite => campsite.Id == id);
        if (campsite == null)
        {
            return Results.NotFound();
        }
        db.Campsites.Remove(campsite);
        db.SaveChanges();
        return Results.NoContent();
    }
    catch (DbUpdateException ex)
    {
        return Results.BadRequest($"Error deleting campsite: {ex.Message}");
    }
});
```

This code catches `DbUpdateException`, which is thrown when there's an error saving changes to the database, such as a constraint violation.

## Checking for Existing Reservations

Before deleting a campsite, you might want to check if there are any active reservations for that campsite. If there are, you might want to prevent the deletion or handle it differently:

```csharp
app.MapDelete("/api/campsites/{id}", (CreekRiverDbContext db, int id) =>
{
    try
    {
        Campsite campsite = db.Campsites
            .Include(c => c.Reservations)
            .SingleOrDefault(campsite => campsite.Id == id);

        if (campsite == null)
        {
            return Results.NotFound();
        }

        // Check for active reservations
        var activeReservations = campsite.Reservations
            .Where(r => r.CheckoutDate > DateTime.Now)
            .ToList();

        if (activeReservations.Any())
        {
            return Results.BadRequest("Cannot delete campsite with active reservations.");
        }

        db.Campsites.Remove(campsite);
        db.SaveChanges();
        return Results.NoContent();
    }
    catch (DbUpdateException ex)
    {
        return Results.BadRequest($"Error deleting campsite: {ex.Message}");
    }
});
```

This code includes the reservations when retrieving the campsite, then checks if there are any active reservations (reservations with a checkout date in the future). If there are, it returns a 400 Bad Request response with an error message.

## The Complete Endpoint

Here's the complete endpoint with validation, error handling, and checking for active reservations:

```csharp
app.MapDelete("/api/campsites/{id}", (CreekRiverDbContext db, int id) =>
{
    try
    {
        Campsite campsite = db.Campsites
            .Include(c => c.Reservations)
            .SingleOrDefault(campsite => campsite.Id == id);

        if (campsite == null)
        {
            return Results.NotFound();
        }

        // Check for active reservations
        var activeReservations = campsite.Reservations
            .Where(r => r.CheckoutDate > DateTime.Now)
            .ToList();

        if (activeReservations.Any())
        {
            return Results.BadRequest("Cannot delete campsite with active reservations.");
        }

        db.Campsites.Remove(campsite);
        db.SaveChanges();
        return Results.NoContent();
    }
    catch (DbUpdateException ex)
    {
        return Results.BadRequest($"Error deleting campsite: {ex.Message}");
    }
});
```

## Conclusion

In this chapter, we've created an API endpoint to delete a campsite from our database using Entity Framework Core. We've learned how to:

1. Create a DELETE endpoint to remove a resource
2. Use Entity Framework Core to delete an entity from the database
3. Return appropriate HTTP responses based on the result of the operation
4. Handle exceptions that might occur during the database operation
5. Check for related entities before deleting an entity

These concepts are fundamental to creating RESTful APIs with ASP.NET Core and Entity Framework Core.

In the next chapter, we'll create an endpoint to update a campsite in the database.

Up Next: [Edit a campsite](./creek-river-put-campsite.md)

## 🔍 Additional Materials

- [ASP.NET Core Minimal APIs](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis)
- [Entity Framework Core - Saving Data](https://docs.microsoft.com/en-us/ef/core/saving/)
- [Entity Framework Core - Cascade Delete](https://docs.microsoft.com/en-us/ef/core/saving/cascade-delete)
- [HTTP DELETE Method](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/DELETE)
- [RESTful API Design](https://docs.microsoft.com/en-us/azure/architecture/best-practices/api-design)