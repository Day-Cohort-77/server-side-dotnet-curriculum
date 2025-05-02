# Harbor Master: Handling POST Requests to Create a Ship

In this chapter, we'll implement a POST endpoint to create a new ship in our Harbor Master API. We'll focus on creating a ship that can be assigned to a dock, demonstrating how to handle relationships between entities.

## Learning Objectives

By the end of this chapter, you should be able to:
- Implement a data access method to insert a new ship with a relationship to a dock
- Create a POST endpoint that handles foreign key relationships
- Validate input data including foreign key constraints
- Return appropriate HTTP responses for created resources

## Implementing the Data Access Method

First, let's add a method to our `DatabaseService` class to create a new ship. Open the `Services/DatabaseService.cs` file and add the following method:

```csharp
// Create a new ship
public async Task<Ship> CreateShipAsync(Ship ship)
{
    using var connection = CreateConnection();
    await connection.OpenAsync();

    using var command = new NpgsqlCommand(
        @"INSERT INTO ships (name, type, dock_id)
          VALUES (@name, @type, @dockId)
          RETURNING id",
        connection);

    command.Parameters.AddWithValue("@name", ship.Name);
    command.Parameters.AddWithValue("@type", ship.Type);

    // Handle null DockId
    if (ship.DockId.HasValue)
    {
        command.Parameters.AddWithValue("@dockId", ship.DockId.Value);
    }
    else
    {
        command.Parameters.AddWithValue("@dockId", DBNull.Value);
    }

    // Execute the command and get the generated ID
    ship.Id = Convert.ToInt32(await command.ExecuteScalarAsync());

    return ship;
}

// Check if a dock exists and has available capacity
public async Task<bool> DockHasAvailableCapacityAsync(int dockId)
{
    using var connection = CreateConnection();
    await connection.OpenAsync();

    // First check if the dock exists
    using var checkDockCommand = new NpgsqlCommand(
        "SELECT 1 FROM docks WHERE id = @dockId",
        connection);
    checkDockCommand.Parameters.AddWithValue("@dockId", dockId);

    var dockExists = await checkDockCommand.ExecuteScalarAsync();
    if (dockExists == null)
    {
        return false; // Dock doesn't exist
    }

    // Then check if the dock has available capacity
    using var capacityCommand = new NpgsqlCommand(
        @"SELECT d.capacity > COUNT(s.id)
          FROM docks d
          LEFT JOIN ships s ON d.id = s.dock_id
          WHERE d.id = @dockId
          GROUP BY d.capacity",
        connection);
    capacityCommand.Parameters.AddWithValue("@dockId", dockId);

    var hasCapacity = await capacityCommand.ExecuteScalarAsync();

    // If the dock has no ships yet, hasCapacity will be null, but the dock has capacity
    return hasCapacity == null || Convert.ToBoolean(hasCapacity);
}
```

We've added two methods:
1. `CreateShipAsync` - Inserts a new ship record with an optional dock assignment
2. `DockHasAvailableCapacityAsync` - Checks if a dock exists and has available capacity

## Creating the POST Endpoint

Now, let's add a POST endpoint to our `ShipEndpoints` class. Open the `Endpoints/ShipEndpoints.cs` file and add the following code:

```csharp
// POST /ships - Create a new ship
app.MapPost("/ships", async (Ship ship, DatabaseService db) =>
{
    try
    {
        // Validate input
        if (string.IsNullOrWhiteSpace(ship.Name))
        {
            return Results.BadRequest("Name is required");
        }

        if (string.IsNullOrWhiteSpace(ship.Type))
        {
            return Results.BadRequest("Type is required");
        }

        // If a dock is specified, check if it exists and has capacity
        if (ship.DockId.HasValue)
        {
            bool dockHasCapacity = await db.DockHasAvailableCapacityAsync(ship.DockId.Value);
            if (!dockHasCapacity)
            {
                return Results.BadRequest("The specified dock does not exist or has no available capacity");
            }
        }

        // Create the ship
        var newShip = await db.CreateShipAsync(ship);

        // Return a 201 Created response with the location of the new resource
        return Results.Created($"/ships/{newShip.Id}", newShip);
    }
    catch (Exception ex)
    {
        return Results.Problem($"An error occurred while creating the ship: {ex.Message}");
    }
})
.WithName("CreateShip")
.WithOpenApi();
```

We've added a POST endpoint that:
1. Maps a POST request to the `/ships` route
2. Accepts a `Ship` object from the request body
3. Validates the input data
4. Checks if the specified dock exists and has available capacity
5. Creates a new ship using the `CreateShipAsync` method
6. Returns a 201 Created response with the location of the new resource
7. Handles exceptions and returns appropriate error responses

## Understanding the POST Endpoint Implementation

Let's take a closer look at how our POST endpoint is implemented:

```csharp
app.MapPost("/ships", async (Ship ship, DatabaseService db) =>
{
    try
    {
        // Validate input
        if (string.IsNullOrWhiteSpace(ship.Name))
        {
            return Results.BadRequest("Name is required");
        }

        if (string.IsNullOrWhiteSpace(ship.Type))
        {
            return Results.BadRequest("Type is required");
        }

        // If a dock is specified, check if it exists and has capacity
        if (ship.DockId.HasValue)
        {
            bool dockHasCapacity = await db.DockHasAvailableCapacityAsync(ship.DockId.Value);
            if (!dockHasCapacity)
            {
                return Results.BadRequest("The specified dock does not exist or has no available capacity");
            }
        }

        // Create the ship
        var newShip = await db.CreateShipAsync(ship);

        // Return a 201 Created response with the location of the new resource
        return Results.Created($"/ships/{newShip.Id}", newShip);
    }
    catch (Exception ex)
    {
        return Results.Problem($"An error occurred while creating the ship: {ex.Message}");
    }
})
.WithName("CreateShip")
.WithOpenApi();
```

This endpoint:
1. Maps a POST request to the `/ships` route
2. Uses model binding to deserialize the request body into a `Ship` object
3. Validates the input data:
   - Checks that the name is not null or empty
   - Checks that the type is not null or empty
4. If a dock is specified (DockId is not null):
   - Checks if the dock exists and has available capacity
   - Returns a 400 Bad Request response if the dock doesn't exist or has no capacity
5. Creates a new ship using the `CreateShipAsync` method
6. Returns a 201 Created response with:
   - A Location header pointing to the new resource
   - The created ship object in the response body
7. Handles exceptions and returns a 500 Internal Server Error response with details

## Testing the POST Endpoint

Let's run the application and test our POST endpoint:

1. Start the API:

```bash
dotnet run
```

2. Open Swagger at `https://localhost:7042/swagger` (or the URL shown in your terminal)

3. Test the `POST /ships` endpoint:
   - Click on the `POST /ships` endpoint
   - Click the "Try it out" button
   - Enter a JSON request body:
     ```json
     {
       "name": "Black Pearl",
       "type": "Galleon",
       "dockId": 1
     }
     ```
   - Click the "Execute" button
   - You should see a 201 Created response with the new ship details, including the generated ID

4. Test creating a ship without assigning it to a dock:
   - Enter a JSON request body:
     ```json
     {
       "name": "Flying Dutchman",
       "type": "Ghost Ship",
       "dockId": null
     }
     ```
   - Click the "Execute" button
   - You should see a 201 Created response with the new ship details

5. Test the validation:
   - Try submitting with an empty name:
     ```json
     {
       "name": "",
       "type": "Galleon",
       "dockId": 1
     }
     ```
   - You should see a 400 Bad Request response with the message "Name is required"

   - Try submitting with a non-existent dock ID:
     ```json
     {
       "name": "HMS Interceptor",
       "type": "Brig",
       "dockId": 999
     }
     ```
   - You should see a 400 Bad Request response with the message "The specified dock does not exist or has no available capacity"

6. Verify the ships were created:
   - Use the `GET /ships` endpoint to see all ships
   - You should see your new ships in the list

## Conclusion

In this chapter, you've learned how to implement a POST endpoint to create a new ship in the Harbor Master API. You've implemented a data access method to insert a new ship record with an optional dock assignment, created a POST endpoint that handles foreign key relationships, validated input data including foreign key constraints, and returned appropriate HTTP responses for created resources.

In the next chapter, you'll implement POST operations for haulers as an exercise.

## Next Steps

[Continue to the next chapter: Implementing POST Operations for Haulers (Exercise)](./harbor-master-post-haulers.md)