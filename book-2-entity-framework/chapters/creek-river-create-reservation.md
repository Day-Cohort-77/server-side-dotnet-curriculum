# Chapter to create a reservation

In this chapter, we'll create an API endpoint to add a new reservation to our database using Entity Framework Core. This will allow clients to book campsites by sending HTTP POST requests to our API.

## Understanding the Reservation Process

Creating a reservation is a bit more complex than creating a campsite because we need to perform several validations:

1. Ensure that the campsite exists
2. Ensure that the user profile exists
3. Validate the check-in and check-out dates
4. Check if the campsite is available for the requested dates
5. Ensure that the reservation duration doesn't exceed the maximum allowed for the campsite type

Let's create an endpoint that handles these validations and creates a new reservation.

## Creating the POST Endpoint for Reservations

Let's create an endpoint to add a new reservation to our database:

1. Open the `Program.cs` file.

2. Add the following endpoint after the existing endpoints:

```csharp
app.MapPost("/api/reservations", (CreekRiverDbContext db, Reservation newRes) =>
{
    db.Reservations.Add(newRes);
    db.SaveChanges();
    return Results.Created($"/api/reservations/{newRes.Id}", newRes);
});
```

This is a basic implementation that adds the new reservation to the database and returns a 201 Created response. However, it doesn't perform any validations. Let's enhance it with proper validation.

## Adding Validation

Let's add validation to ensure that the reservation is valid:

```csharp
app.MapPost("/api/reservations", (CreekRiverDbContext db, Reservation newRes) =>
{
    try
    {
        // Validate that the campsite exists
        var campsite = db.Campsites
            .Include(c => c.CampsiteType)
            .SingleOrDefault(c => c.Id == newRes.CampsiteId);

        if (campsite == null)
        {
            return Results.BadRequest($"Campsite with ID {newRes.CampsiteId} does not exist.");
        }

        // Validate that the user profile exists
        var userProfile = db.UserProfiles.Find(newRes.UserProfileId);
        if (userProfile == null)
        {
            return Results.BadRequest($"UserProfile with ID {newRes.UserProfileId} does not exist.");
        }

        // Validate check-in and check-out dates
        if (newRes.CheckinDate >= newRes.CheckoutDate)
        {
            return Results.BadRequest("Check-out date must be after check-in date.");
        }

        // Calculate reservation duration
        var duration = (newRes.CheckoutDate - newRes.CheckinDate).Days;

        // Validate reservation duration against campsite type's maximum
        if (duration > campsite.CampsiteType.MaxReservationDays)
        {
            return Results.BadRequest($"Reservation duration ({duration} days) exceeds the maximum allowed for this campsite type ({campsite.CampsiteType.MaxReservationDays} days).");
        }

        // Check if the campsite is available for the requested dates
        var conflictingReservation = db.Reservations
            .Where(r => r.CampsiteId == newRes.CampsiteId)
            .Where(r =>
                (newRes.CheckinDate >= r.CheckinDate && newRes.CheckinDate < r.CheckoutDate) || // Check-in date falls within an existing reservation
                (newRes.CheckoutDate > r.CheckinDate && newRes.CheckoutDate <= r.CheckoutDate) || // Check-out date falls within an existing reservation
                (newRes.CheckinDate <= r.CheckinDate && newRes.CheckoutDate >= r.CheckoutDate)) // New reservation completely encompasses an existing reservation
            .FirstOrDefault();

        if (conflictingReservation != null)
        {
            return Results.BadRequest($"Campsite is not available for the requested dates. There is a conflicting reservation from {conflictingReservation.CheckinDate.ToShortDateString()} to {conflictingReservation.CheckoutDate.ToShortDateString()}.");
        }

        // Add the reservation to the database
        db.Reservations.Add(newRes);
        db.SaveChanges();

        return Results.Created($"/api/reservations/{newRes.Id}", newRes);
    }
    catch (DbUpdateException ex)
    {
        return Results.BadRequest($"Error creating reservation: {ex.Message}");
    }
});
```

Let's break down this code:

1. **Validate Campsite**: We check if the campsite exists and include its campsite type for later validation.

2. **Validate User Profile**: We check if the user profile exists.

3. **Validate Dates**: We ensure that the check-out date is after the check-in date.

4. **Validate Duration**: We calculate the reservation duration and ensure it doesn't exceed the maximum allowed for the campsite type.

5. **Check Availability**: We check if the campsite is available for the requested dates by looking for conflicting reservations.

6. **Create Reservation**: If all validations pass, we add the reservation to the database and return a 201 Created response.

7. **Handle Exceptions**: We catch any database update exceptions and return a 400 Bad Request response with an error message.


## Adding a Cancellation Endpoint

In addition to creating reservations, you might want to allow users to cancel their reservations. Let's create an endpoint to delete a reservation:

```csharp
app.MapDelete("/api/reservations/{id}", (CreekRiverDbContext db, int id) =>
{
    try
    {
        Reservation reservation = db.Reservations.SingleOrDefault(r => r.Id == id);
        if (reservation == null)
        {
            return Results.NotFound();
        }

        // Check if the reservation has already started
        if (reservation.CheckinDate <= DateTime.Now)
        {
            return Results.BadRequest("Cannot cancel a reservation that has already started.");
        }

        db.Reservations.Remove(reservation);
        db.SaveChanges();
        return Results.NoContent();
    }
    catch (DbUpdateException ex)
    {
        return Results.BadRequest($"Error cancelling reservation: {ex.Message}");
    }
});
```

This endpoint deletes a reservation by its ID, but only if the reservation hasn't already started.

## Testing the Endpoints

Now that we've created our endpoints, let's test them:

1. Run the application with `dotnet run` or by pressing F5 in Visual Studio.

2. Use a tool like Postman to send a POST request to `https://localhost:<port>/api/reservations` with a JSON payload like:

```json
{
  "campsiteId": 1,
  "userProfileId": 1,
  "checkinDate": "2023-07-15",
  "checkoutDate": "2023-07-18"
}
```

3. You should receive a 201 Created response with the newly created reservation in the response body if all validations pass, or a 400 Bad Request response with an error message if any validation fails.

4. To test the cancellation endpoint, send a DELETE request to `https://localhost:<port>/api/reservations/1` (replace `1` with the ID of a reservation you want to cancel).

## Conclusion

In this chapter, we've created API endpoints to create and cancel reservations using Entity Framework Core. We've learned how to:

1. Create a POST endpoint to add a new reservation
2. Validate the reservation data to ensure it's valid
3. Check if the campsite is available for the requested dates
5. Create a DELETE endpoint to cancel a reservation

These concepts are fundamental to creating a reservation system with ASP.NET Core and Entity Framework Core.

In the next chapters, you might want to explore more advanced topics such as:

1. Adding authentication and authorization to protect the API
2. Implementing a payment system for reservations
3. Adding email notifications for reservation confirmations and reminders
4. Creating a user interface for the reservation system

## 🔍 Additional Materials

- [Entity Framework Core - Saving Data](https://docs.microsoft.com/en-us/ef/core/saving/)
- [Entity Framework Core - Querying Data](https://docs.microsoft.com/en-us/ef/core/querying/)
- [ASP.NET Core Minimal APIs](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis)
- [Data Validation in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/mvc/models/validation)
- [RESTful API Design](https://docs.microsoft.com/en-us/azure/architecture/best-practices/api-design)