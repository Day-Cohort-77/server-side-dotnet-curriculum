# Harbor Master: Handling DELETE Requests for Ships

In this chapter, we'll implement a DELETE endpoint to remove ships from our Harbor Master API. We'll learn how to handle DELETE requests and remove records from the PostgreSQL database.

## Learning Objectives

By the end of this chapter, you should be able to:
- Implement a data access method to delete a record from PostgreSQL
- Create a DELETE endpoint in a minimal API
- Handle route parameters in DELETE requests
- Return appropriate HTTP responses for deleted resources

## Implementing the Data Access Method

First, let's add a method to our `DatabaseService` class to delete a ship. Open the `Services/DatabaseService.cs` file and add the following method:

```csharp
// Delete a ship
public async Task<bool> DeleteShipAsync(int id)
{
    using var connection = CreateConnection();
    await connection.OpenAsync();

    using var command = new NpgsqlCommand(
        "DELETE FROM ships WHERE id = @id",
        connection);

    command.Parameters.AddWithValue("@id", id);

    int rowsAffected = await command.ExecuteNonQueryAsync();

    return rowsAffected > 0;
}
```

This method:
1. Opens a connection to the database
2. Creates a SQL command to delete a ship record by ID
3. Adds a parameter for the ship's ID
4. Executes the command and gets the number of affected rows
5. Returns a boolean indicating whether a record was deleted (true) or not (false)

## Creating the DELETE Endpoint

Now, let's add a DELETE endpoint to our `ShipEndpoints` class. Open the `Endpoints/ShipEndpoints.cs` file and add the following code:

```csharp
// DELETE /ships/{id} - Delete a ship
app.MapDelete("/ships/{id}", async (int id, DatabaseService db) =>
{
    try
    {
        bool deleted = await db.DeleteShipAsync(id);

        if (deleted)
        {
            // Return a 204 No Content response
            return Results.NoContent();
        }
        else
        {
            // Return a 404 Not Found response
            return Results.NotFound();
        }
    }
    catch (Exception ex)
    {
        return Results.Problem($"An error occurred while deleting the ship: {ex.Message}");
    }
});
```

We've added a DELETE endpoint that:
1. Maps a DELETE request to the `/ships/{id}` route
2. Accepts an ID parameter from the route
3. Deletes the ship using the `DeleteShipAsync` method
4. Returns a 204 No Content response if the ship was deleted
5. Returns a 404 Not Found response if the ship wasn't found
6. Handles exceptions and returns appropriate error responses

## Understanding the DELETE Endpoint Implementation

Let's take a closer look at how our DELETE endpoint is implemented:

```csharp
app.MapDelete("/ships/{id}", async (int id, DatabaseService db) =>
{
    try
    {
        bool deleted = await db.DeleteShipAsync(id);

        if (deleted)
        {
            // Return a 204 No Content response
            return Results.NoContent();
        }
        else
        {
            // Return a 404 Not Found response
            return Results.NotFound();
        }
    }
    catch (Exception ex)
    {
        return Results.Problem($"An error occurred while deleting the ship: {ex.Message}");
    }
});
```

This endpoint:
1. Maps a DELETE request to the `/ships/{id}` route, where `{id}` is a route parameter
2. Uses dependency injection to get an instance of `DatabaseService`
3. Calls the `DeleteShipAsync` method to delete a ship by ID
4. Returns a 204 No Content status if the ship was deleted
   - 204 No Content is the standard response for successful DELETE operations
   - It indicates that the request was successful but there's no content to return
5. Returns a 404 Not Found status if the ship wasn't found
6. Handles exceptions and returns a 500 Internal Server Error response with details

## Testing the DELETE Endpoint

Let's run the application and test our DELETE endpoint:

1. Restart your debugger

2. Make a GET request to `/ships` and find the `id` of one you want to delete

3. Make a DELETE request to `/ships/{id}` endpoint:
   - You should see a 204 No Content response

4. Verify the ship was deleted:
   - Use the GET `/ships` endpoint again
   - The ship with the deleted ID should no longer be in the list

5. Try deleting a non-existent ship:
   - Use the DELETE `/ships/{id}` endpoint with an ID that doesn't exist (e.g., 999)
   - You should see a 404 Not Found response

## Conclusion

In this chapter, you've learned how to implement a DELETE endpoint to remove ships from the Harbor Master API. You've implemented a data access method to delete a record from PostgreSQL, created a DELETE endpoint in a minimal API, handled route parameters in DELETE requests, and returned appropriate HTTP responses for deleted resources.

In the next chapter, you'll implement DELETE operations for docks and haulers as an exercise.

## Next Steps

[Continue to the next chapter: Implementing DELETE Operations for Docks and Haulers (Exercise)](./harbor-master-delete-exercise.md)