# Harbor Master: Handling PUT Requests to Modify Docks

In this chapter, we'll implement a PUT endpoint to modify existing docks in our Harbor Master API. We'll learn how to handle PUT requests, validate input data, and update existing records in the PostgreSQL database.

## Learning Objectives

By the end of this chapter, you should be able to:
- Implement a data access method to update an existing record in PostgreSQL
- Create a PUT endpoint in a minimal API
- Handle request body data for updates
- Return appropriate HTTP responses for updated resources
- Validate that updates don't violate business rules

## Understanding PUT Requests

In REST APIs, PUT requests are used to update existing resources. When we make a PUT request, we're asking the server to replace the current version of a resource with the new version we're providing.

For our dock update endpoint, we'll:
1. Receive a PUT request with the dock ID in the URL and the updated dock data in the request body
2. Check if the dock exists
3. Validate the updated data
4. Update the dock in the database
5. Return an appropriate response

## Implementing the Data Access Method

First, let's add a method to our `DatabaseService` class to update an existing dock. Open the `Services/DatabaseService.cs` file and add the following method:

```csharp
// Update an existing dock
public async Task<Dock> UpdateDockAsync(Dock dock)
{
    using var connection = CreateConnection();
    await connection.OpenAsync();

    using var command = new NpgsqlCommand(
        @"UPDATE docks
          SET location = @location,
              capacity = @capacity
          WHERE id = @id",
        connection);

    command.Parameters.AddWithValue("@id", dock.Id);
    command.Parameters.AddWithValue("@location", dock.Location);
    command.Parameters.AddWithValue("@capacity", dock.Capacity);

    // Execute the command
    await command.ExecuteNonQueryAsync();

    // Retrieve and return the updated dock
    return await GetDockByIdAsync(dock.Id);
}
```

This method:
1. Opens a connection to the database
2. Creates a SQL command to update a dock record
3. Adds parameters for the dock's ID, location, and capacity
4. Executes the command
5. Retrieves and returns the updated dock

## Checking Capacity Constraints

Before updating a dock's capacity, we need to ensure that the new capacity is not less than the number of ships currently docked there. Let's add a method to check this:

```csharp
// Check if a dock has enough capacity for its current ships
public async Task<bool> CanUpdateDockCapacityAsync(int dockId, int newCapacity)
{
    using var connection = CreateConnection();
    await connection.OpenAsync();

    using var command = new NpgsqlCommand(
        @"SELECT COUNT(*)
          FROM ships
          WHERE dock_id = @dockId",
        connection);

    command.Parameters.AddWithValue("@dockId", dockId);

    // Get the number of ships at this dock
    int shipCount = Convert.ToInt32(await command.ExecuteScalarAsync());

    // Return true if the new capacity is sufficient
    return newCapacity >= shipCount;
}
```

This method:
1. Counts the number of ships currently assigned to the dock
2. Compares this count to the proposed new capacity
3. Returns true if the new capacity is sufficient, false otherwise

## Creating the PUT Endpoint

Now, let's add a PUT endpoint to our `DockEndpoints` class. Open the `Endpoints/DockEndpoints.cs` file and add the following code:

```csharp
// PUT /docks/{id} - Update a dock
app.MapPut("/docks/{id}", async (int id, Dock updatedDock, DatabaseService db) =>
{
    try
    {
        // Check if the dock exists
        var existingDock = await db.GetDockByIdAsync(id);
        if (existingDock == null)
        {
            return Results.NotFound($"Dock with ID {id} not found");
        }

        // Validate input
        if (string.IsNullOrWhiteSpace(updatedDock.Location))
        {
            return Results.BadRequest("Location is required");
        }

        if (updatedDock.Capacity <= 0)
        {
            return Results.BadRequest("Capacity must be greater than zero");
        }

        // Check if the new capacity is sufficient for current ships
        if (updatedDock.Capacity < existingDock.Capacity)
        {
            bool canUpdate = await db.CanUpdateDockCapacityAsync(id, updatedDock.Capacity);
            if (!canUpdate)
            {
                return Results.BadRequest("Cannot reduce capacity below the number of ships currently at this dock");
            }
        }

        // Set the ID from the route parameter
        updatedDock.Id = id;

        // Update the dock
        var result = await db.UpdateDockAsync(updatedDock);

        // Return the updated dock
        return Results.Ok(result);
    }
    catch (Exception ex)
    {
        return Results.Problem($"An error occurred while updating the dock: {ex.Message}");
    }
});
```

We've added a PUT endpoint that:
1. Maps a PUT request to the `/docks/{id}` route
2. Accepts an ID parameter from the route and a `Dock` object from the request body
3. Checks if the dock exists
4. Validates the input data
5. Checks if the new capacity is sufficient for the current ships
6. Updates the dock using the `UpdateDockAsync` method
7. Returns a 200 OK response with the updated dock
8. Handles exceptions and returns appropriate error responses

## Understanding the PUT Endpoint Implementation

Let's take a closer look at how our PUT endpoint is implemented:

```csharp
app.MapPut("/docks/{id}", async (int id, Dock updatedDock, DatabaseService db) =>
{
    try
    {
        // Check if the dock exists
        var existingDock = await db.GetDockByIdAsync(id);
        if (existingDock == null)
        {
            return Results.NotFound($"Dock with ID {id} not found");
        }

        // Validate input
        if (string.IsNullOrWhiteSpace(updatedDock.Location))
        {
            return Results.BadRequest("Location is required");
        }

        if (updatedDock.Capacity <= 0)
        {
            return Results.BadRequest("Capacity must be greater than zero");
        }

        // Check if the new capacity is sufficient for current ships
        if (updatedDock.Capacity < existingDock.Capacity)
        {
            bool canUpdate = await db.CanUpdateDockCapacityAsync(id, updatedDock.Capacity);
            if (!canUpdate)
            {
                return Results.BadRequest("Cannot reduce capacity below the number of ships currently at this dock");
            }
        }

        // Set the ID from the route parameter
        updatedDock.Id = id;

        // Update the dock
        var result = await db.UpdateDockAsync(updatedDock);

        // Return the updated dock
        return Results.Ok(result);
    }
    catch (Exception ex)
    {
        return Results.Problem($"An error occurred while updating the dock: {ex.Message}");
    }
});
```

This endpoint:
1. Maps a PUT request to the `/docks/{id}` route, where `{id}` is a route parameter
2. Uses model binding to deserialize the request body into a `Dock` object
3. Checks if the dock exists
4. Validates the input data:
   - Checks that the location is not null or empty
   - Checks that the capacity is greater than zero
5. Checks if the new capacity is sufficient for the current ships:
   - If the new capacity is less than the current capacity, it checks if there are fewer ships than the new capacity
   - If not, it returns a 400 Bad Request response
6. Sets the ID from the route parameter
7. Updates the dock using the `UpdateDockAsync` method
8. Returns a 200 OK response with the updated dock
9. Handles exceptions and returns a 500 Internal Server Error response with details

## Testing the PUT Endpoint

Let's run the application and test our PUT endpoint:

1. Restart your debugger

2. Open your API client tool

3. Test the `PUT /docks/{id}` endpoint:
   - Choose an existing dock ID
   - Enter a JSON request body:
     ```json
     {
       "location": "Updated East Harbor",
       "capacity": 15
     }
     ```
   - You should see a 200 OK response with the updated dock details

4. Test the validation:
   - Try submitting with an empty location:
     ```json
     {
       "location": "",
       "capacity": 15
     }
     ```
   - You should see a 400 Bad Request response with the message "Location is required"

   - Try submitting with a negative capacity:
     ```json
     {
       "location": "East Harbor",
       "capacity": -5
     }
     ```
   - You should see a 400 Bad Request response with the message "Capacity must be greater than zero"

5. Test the capacity constraint:
   - Find a dock with ships assigned to it
   - Try to update its capacity to a value less than the number of ships
   - You should see a 400 Bad Request response with the message "Cannot reduce capacity below the number of ships currently at this dock"

6. Verify the dock was updated:
   - Use the `GET /docks/{id}` endpoint to see the updated dock
   - You should see the changes you made

## Conclusion

In this chapter, you've learned how to implement a PUT endpoint to update an existing dock in the Harbor Master API. You've implemented a data access method to update a record in PostgreSQL, created a PUT endpoint in a minimal API, handled request body data for updates, and returned appropriate HTTP responses for updated resources.

You've also learned how to validate that updates don't violate business rules, such as ensuring that a dock's capacity is not reduced below the number of ships currently docked there.

In the next chapter, you'll implement PUT operations for ships and haulers as an exercise.

## Next Steps

[Continue to the next chapter: Implementing PUT Operations for Ships and Haulers (Exercise)](./harbor-master-modify-ships-haulers.md)