# Harbor Master: Handling GET Operations for Ships

In this chapter, we'll implement GET operations to retrieve ship data from our PostgreSQL database. We'll create endpoints to get all ships and to get a single ship by ID.

## Learning Objectives

By the end of this chapter, you should be able to:
- Implement data access methods to retrieve data from PostgreSQL
- Create GET endpoints in a minimal API
- Handle parameters in API routes
- Return appropriate HTTP responses

## Implementing Data Access Methods

First, let's add methods to our `DatabaseService` class to retrieve ship data from the database. Open the `Services/DatabaseService.cs` file and add the following methods:

```csharp
// Get all ships
public async Task<List<Ship>> GetAllShipsAsync()
{
    var ships = new List<Ship>();

    using var connection = CreateConnection();
    await connection.OpenAsync();

    using var command = new NpgsqlCommand("SELECT id, name, type, dock_id FROM ships", connection);
    using var reader = await command.ExecuteReaderAsync();

    while (await reader.ReadAsync())
    {
        ships.Add(new Ship
        {
            Id = reader.GetInt32(0),
            Name = reader.GetString(1),
            Type = reader.GetString(2),
            DockId = reader.IsDBNull(3) ? null : reader.GetInt32(3)
        });
    }

    return ships;
}

// Get ship by ID
public async Task<Ship?> GetShipByIdAsync(int id)
{
    using var connection = CreateConnection();
    await connection.OpenAsync();

    using var command = new NpgsqlCommand(
        "SELECT id, name, type, dock_id FROM ships WHERE id = @id",
        connection);
    command.Parameters.AddWithValue("@id", id);

    using var reader = await command.ExecuteReaderAsync();

    if (await reader.ReadAsync())
    {
        return new Ship
        {
            Id = reader.GetInt32(0),
            Name = reader.GetString(1),
            Type = reader.GetString(2),
            DockId = reader.IsDBNull(3) ? null : reader.GetInt32(3)
        };
    }

    return null;
}
```

These methods:
1. `GetAllShipsAsync` - Retrieves all ships from the database
2. `GetShipByIdAsync` - Retrieves a single ship by its ID

## Creating Endpoint Extension Methods

Now, let's create an extension method class to define our ship endpoints. Create a new file called `Endpoints/ShipEndpoints.cs`:

```csharp
using HarborMaster.Models;
using HarborMaster.Services;

namespace HarborMaster.Endpoints
{
    public static class ShipEndpoints
    {
        public static void MapShipEndpoints(this WebApplication app)
        {
            // GET /ships - Get all ships
            app.MapGet("/ships", async (DatabaseService db) =>
                await db.GetAllShipsAsync())
                .WithName("GetAllShips")
                .WithOpenApi();

            // GET /ships/{id} - Get a ship by ID
            app.MapGet("/ships/{id}", async (int id, DatabaseService db) =>
            {
                var ship = await db.GetShipByIdAsync(id);
                return ship != null ? Results.Ok(ship) : Results.NotFound();
            })
            .WithName("GetShipById")
            .WithOpenApi();
        }
    }
}
```

This class defines two endpoints:
1. `GET /ships` - Returns all ships
2. `GET /ships/{id}` - Returns a single ship by ID

## Updating Program.cs

Now, let's update the `Program.cs` file to use our ship endpoints:

```csharp
using HarborMaster.Services;
using HarborMaster.Endpoints;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
// Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Add database service
builder.Services.AddSingleton<DatabaseService>();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

// Initialize and seed the database
using (var scope = app.Services.CreateScope())
{
    var dbService = scope.ServiceProvider.GetRequiredService<DatabaseService>();
    await dbService.InitializeDatabaseAsync();
    await dbService.SeedDatabaseAsync();
}

// Define API endpoints
app.MapGet("/", () => "Welcome to Harbor Master API!");

// Map ship endpoints
app.MapShipEndpoints();

app.Run();
```

We've added the `using HarborMaster.Endpoints;` directive and the call to `app.MapShipEndpoints()` to register our ship endpoints.

## Understanding the Endpoint Implementation

Let's take a closer look at how our endpoints are implemented:

### GET /ships - Get All Ships

```csharp
app.MapGet("/ships", async (DatabaseService db) =>
    await db.GetAllShipsAsync())
    .WithName("GetAllShips")
    .WithOpenApi();
```

This endpoint:
1. Maps a GET request to the `/ships` route
2. Uses dependency injection to get an instance of `DatabaseService`
3. Calls the `GetAllShipsAsync` method to retrieve all ships
4. Returns the list of ships as a JSON response
5. Uses `.WithName()` to give the endpoint a name for Swagger documentation
6. Uses `.WithOpenApi()` to include the endpoint in the OpenAPI specification

### GET /ships/{id} - Get Ship by ID

```csharp
app.MapGet("/ships/{id}", async (int id, DatabaseService db) =>
{
    var ship = await db.GetShipByIdAsync(id);
    return ship != null ? Results.Ok(ship) : Results.NotFound();
})
.WithName("GetShipById")
.WithOpenApi();
```

This endpoint:
1. Maps a GET request to the `/ships/{id}` route, where `{id}` is a route parameter
2. Uses dependency injection to get an instance of `DatabaseService`
3. Calls the `GetShipByIdAsync` method to retrieve a ship by ID
4. Returns the ship as a JSON response with a 200 OK status if found
5. Returns a 404 Not Found status if the ship doesn't exist
6. Uses `.WithName()` and `.WithOpenApi()` for documentation

## Testing the Endpoints

Let's run the application and test our endpoints:

1. Start the API:

```bash
dotnet run
```

2. Open Swagger at `https://localhost:7042/swagger` (or the URL shown in your terminal)

3. Test the `GET /ships` endpoint:
   - Click on the `GET /ships` endpoint
   - Click the "Try it out" button
   - Click the "Execute" button
   - You should see a response with the list of ships

4. Test the `GET /ships/{id}` endpoint:
   - Click on the `GET /ships/{id}` endpoint
   - Click the "Try it out" button
   - Enter an ID (e.g., 1)
   - Click the "Execute" button
   - You should see a response with the ship details

5. Try the `GET /ships/{id}` endpoint with an invalid ID:
   - Enter an ID that doesn't exist (e.g., 999)
   - Click the "Execute" button
   - You should see a 404 Not Found response

## Conclusion

In this chapter, you've learned how to implement GET operations to retrieve ship data from a PostgreSQL database. You've created endpoints to get all ships and to get a single ship by ID, and you've learned how to handle route parameters and return appropriate HTTP responses.

In the next chapter, you'll implement GET operations for docks as an exercise.

## Next Steps

[Continue to the next chapter: Implementing GET Operations for Docks (Exercise)](./harbor-master-get-docks.md)