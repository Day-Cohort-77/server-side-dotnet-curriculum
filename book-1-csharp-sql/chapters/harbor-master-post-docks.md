# Harbor Master: Handling POST Requests to Create a Dock

In this chapter, we'll implement a POST endpoint to create a new dock in our Harbor Master API. We'll learn how to handle POST requests, validate input data, and insert new records into the PostgreSQL database.

## Learning Objectives

By the end of this chapter, you should be able to:
- Implement a data access method to insert a new record into PostgreSQL
- Create a POST endpoint in a minimal API
- Handle request body data
- Return appropriate HTTP responses for created resources

## Implementing the Data Access Method

First, let's add a method to our `DatabaseService` class to create a new dock. Open the `Services/DatabaseService.cs` file and add the following method:

```csharp
// Create a new dock
public async Task<Dock> CreateDockAsync(Dock dock)
{
    using var connection = CreateConnection();
    await connection.OpenAsync();

    using var command = new NpgsqlCommand(
        @"INSERT INTO docks (location, capacity)
          VALUES (@location, @capacity)
          RETURNING id",
        connection);

    command.Parameters.AddWithValue("@location", dock.Location);
    command.Parameters.AddWithValue("@capacity", dock.Capacity);

    // Execute the command and get the generated ID
    dock.Id = Convert.ToInt32(await command.ExecuteScalarAsync());

    return dock;
}
```

This method:
1. Opens a connection to the database
2. Creates a SQL command to insert a new dock record
3. Adds parameters for the dock's location and capacity
4. Executes the command and retrieves the generated ID
5. Updates the dock object with the new ID
6. Returns the updated dock object

## Creating the POST Endpoint

Now, let's add a POST endpoint to our `DockEndpoints` class. If you completed the previous exercise, you should already have a `Endpoints/DockEndpoints.cs` file. If not, create it now and add the following code:

```csharp
using HarborMaster.Models;
using HarborMaster.Services;

namespace HarborMaster.Endpoints
{
    public static class DockEndpoints
    {
        public static void MapDockEndpoints(this WebApplication app)
        {
            // GET /docks - Get all docks
            app.MapGet("/docks", async (DatabaseService db) =>
                await db.GetAllDocksAsync())
                .WithName("GetAllDocks")
                .WithOpenApi();

            // GET /docks/{id} - Get a dock by ID
            app.MapGet("/docks/{id}", async (int id, DatabaseService db) =>
            {
                var dock = await db.GetDockByIdAsync(id);
                return dock != null ? Results.Ok(dock) : Results.NotFound();
            })
            .WithName("GetDockById")
            .WithOpenApi();

            // POST /docks - Create a new dock
            app.MapPost("/docks", async (Dock dock, DatabaseService db) =>
            {
                try
                {
                    // Validate input
                    if (string.IsNullOrWhiteSpace(dock.Location))
                    {
                        return Results.BadRequest("Location is required");
                    }

                    if (dock.Capacity <= 0)
                    {
                        return Results.BadRequest("Capacity must be greater than zero");
                    }

                    // Create the dock
                    var newDock = await db.CreateDockAsync(dock);

                    // Return a 201 Created response with the location of the new resource
                    return Results.Created($"/docks/{newDock.Id}", newDock);
                }
                catch (Exception ex)
                {
                    return Results.Problem($"An error occurred while creating the dock: {ex.Message}");
                }
            })
            .WithName("CreateDock")
            .WithOpenApi();
        }
    }
}
```

We've added a new POST endpoint that:
1. Maps a POST request to the `/docks` route
2. Accepts a `Dock` object from the request body
3. Validates the input data
4. Creates a new dock using the `CreateDockAsync` method
5. Returns a 201 Created response with the location of the new resource
6. Handles exceptions and returns appropriate error responses

## Understanding the POST Endpoint Implementation

Let's take a closer look at how our POST endpoint is implemented:

```csharp
app.MapPost("/docks", async (Dock dock, DatabaseService db) =>
{
    try
    {
        // Validate input
        if (string.IsNullOrWhiteSpace(dock.Location))
        {
            return Results.BadRequest("Location is required");
        }

        if (dock.Capacity <= 0)
        {
            return Results.BadRequest("Capacity must be greater than zero");
        }

        // Create the dock
        var newDock = await db.CreateDockAsync(dock);

        // Return a 201 Created response with the location of the new resource
        return Results.Created($"/docks/{newDock.Id}", newDock);
    }
    catch (Exception ex)
    {
        return Results.Problem($"An error occurred while creating the dock: {ex.Message}");
    }
})
.WithName("CreateDock")
.WithOpenApi();
```

This endpoint:
1. Maps a POST request to the `/docks` route
2. Uses model binding to deserialize the request body into a `Dock` object
3. Validates the input data:
   - Checks that the location is not null or empty
   - Checks that the capacity is greater than zero
4. Creates a new dock using the `CreateDockAsync` method
5. Returns a 201 Created response with:
   - A Location header pointing to the new resource
   - The created dock object in the response body
6. Handles exceptions and returns a 500 Internal Server Error response with details

## Testing the POST Endpoint

Let's run the application and test our POST endpoint:

1. Start the API:

```bash
dotnet run
```

2. Open Swagger at `https://localhost:7042/swagger` (or the URL shown in your terminal)

3. Test the `POST /docks` endpoint:
   - Click on the `POST /docks` endpoint
   - Click the "Try it out" button
   - Enter a JSON request body:
     ```json
     {
       "location": "West Harbor",
       "capacity": 10
     }
     ```
   - Click the "Execute" button
   - You should see a 201 Created response with the new dock details, including the generated ID

4. Test the validation:
   - Try submitting with an empty location:
     ```json
     {
       "location": "",
       "capacity": 10
     }
     ```
   - You should see a 400 Bad Request response with the message "Location is required"

   - Try submitting with a negative capacity:
     ```json
     {
       "location": "West Harbor",
       "capacity": -5
     }
     ```
   - You should see a 400 Bad Request response with the message "Capacity must be greater than zero"

5. Verify the dock was created:
   - Use the `GET /docks` endpoint to see all docks
   - You should see your new dock in the list

## Conclusion

In this chapter, you've learned how to implement a POST endpoint to create a new dock in the Harbor Master API. You've implemented a data access method to insert a new record into PostgreSQL, created a POST endpoint in a minimal API, handled request body data, and returned appropriate HTTP responses for created resources.

In the next chapter, we'll learn how to handle POST requests to create a ship that is assigned to one of the docks.

## Next Steps

[Continue to the next chapter: Handling POST Requests to Create a Ship](./harbor-master-post-ships.md)