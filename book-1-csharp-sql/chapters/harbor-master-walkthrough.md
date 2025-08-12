# Harbor Master Project Walkthrough

In this chapter, we'll build the Harbor Master project from scratch. This will be a step-by-step guide to creating a minimal API that connects to a PostgreSQL database, handles database operations without Entity Framework Core, and processes various types of client requests.

## Learning Objectives

By the end of this chapter, you should be able to:
- Create a minimal API project in ASP.NET Core
- Connect to a PostgreSQL database using Npgsql directly (without EF Core)
- Seed a database with initial data using SQL scripts
- Handle GET requests with and without parameters
- Process POST requests from clients
- Implement query string parameter handling

## Prerequisites

Before starting this project, ensure you have:
- .NET 8 SDK installed
- PostgreSQL installed and running
- SQLTools extension
- Basic understanding of C# and SQL

## Step 1: Creating the Minimal API Project

Let's start by creating a new minimal API project:

1. Open a terminal or command prompt
2. Navigate to your desired project directory
3. Run the following command:

```bash
dotnet new webapi -minimal -o HarborMaster
```

4. Navigate to the project directory:

```bash
cd HarborMaster
```

5. Open the project in your preferred code editor:

```bash
code .  # If using Visual Studio Code
```

## Step 2: Setting Up the Project Structure

Let's organize our project by creating the necessary folders and files:

1. Create the following folders:
   - `Models` - For our data models
   - `Services` - For database interaction
   - `Endpoints` - For API endpoint definitions

2. Add the Npgsql package to connect to PostgreSQL:

```bash
dotnet add package Npgsql
```

## Step 3: Creating the Data Models

Let's define our data models. Create the following files:

1. Create `Models/Ship.cs`:

```csharp
namespace HarborMaster.Models
{
    public class Ship
    {
        public int Id { get; set; }
        public string Name { get; set; } = string.Empty;
        public string Type { get; set; } = string.Empty;
        public int? DockId { get; set; }
    }
}
```

2. Create `Models/Dock.cs`:

```csharp
namespace HarborMaster.Models
{
    public class Dock
    {
        public int Id { get; set; }
        public string Location { get; set; } = string.Empty;
        public int Capacity { get; set; }
    }
}
```

3. Create `Models/Hauler.cs`:

```csharp
namespace HarborMaster.Models
{
    public class Hauler
    {
        public int Id { get; set; }
        public string Name { get; set; } = string.Empty;
        public int Capacity { get; set; }
    }
}
```

## Step 4: Setting Up Database Connection

Now, let's set up the database connection:

1. Update `appsettings.json` to include the connection string:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Host=localhost;Database=harbormaster;Username=postgres;Password=your_password"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

Replace `your_password` with your actual PostgreSQL password.

2. Create a database service to handle database operations. Create `Services/DatabaseService.cs`:

```csharp
using Npgsql;
using HarborMaster.Models;
using System.Data;

namespace HarborMaster.Services
{
    public class DatabaseService
    {
        private readonly string _connectionString;

        public DatabaseService(IConfiguration configuration)
        {
            _connectionString = configuration.GetConnectionString("DefaultConnection") ??
                throw new InvalidOperationException("Connection string 'DefaultConnection' not found.");
        }

        public NpgsqlConnection CreateConnection()
        {
            return new NpgsqlConnection(_connectionString);
        }

        // Helper method to execute non-query SQL commands
        public async Task ExecuteNonQueryAsync(string sql, Dictionary<string, object>? parameters = null)
        {
            using var connection = CreateConnection();
            await connection.OpenAsync();

            using var command = new NpgsqlCommand(sql, connection);
            if (parameters != null)
            {
                foreach (var param in parameters)
                {
                    command.Parameters.AddWithValue(param.Key, param.Value);
                }
            }

            await command.ExecuteNonQueryAsync();
        }
    }
}
```

## Step 5: Creating the Database Schema

Let's create the database schema. Create a file called `database-setup.sql` in the project root:

```sql
-- Drop tables if they exist
DROP TABLE IF EXISTS ships;
DROP TABLE IF EXISTS docks;
DROP TABLE IF EXISTS haulers;

-- Create tables
CREATE TABLE docks (
    id SERIAL PRIMARY KEY,
    location VARCHAR(100) NOT NULL,
    capacity INTEGER NOT NULL
);

CREATE TABLE haulers (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    capacity INTEGER NOT NULL
);

CREATE TABLE ships (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    type VARCHAR(100) NOT NULL,
    dock_id INTEGER,
    hauler_id INTEGER,
    FOREIGN KEY (dock_id) REFERENCES docks(id) ON DELETE SET NULL,
    FOREIGN KEY (hauler_id) REFERENCES haulers(id) ON DELETE SET NULL
);
```

Now, let's add a method to our `DatabaseService` to initialize the database:

```csharp
public async Task InitializeDatabaseAsync()
{
    // First, create the database if it doesn't exist
    using var connection = new NpgsqlConnection(_connectionString.Replace("Database=harbormaster", "Database=postgres"));
    await connection.OpenAsync();

    // Check if database exists
    using var checkCommand = new NpgsqlCommand(
        "SELECT 1 FROM pg_database WHERE datname = 'harbormaster'",
        connection);
    var exists = await checkCommand.ExecuteScalarAsync();

    if (exists == null)
    {
        // Create the database
        using var createDbCommand = new NpgsqlCommand(
            "CREATE DATABASE harbormaster",
            connection);
        await createDbCommand.ExecuteNonQueryAsync();
    }

    // Now connect to the harbormaster database and create tables
    string sql = File.ReadAllText("database-setup.sql");
    await ExecuteNonQueryAsync(sql);
}
```

## Step 6: Seeding the Database

Now, let's add a method to seed the database with initial data. Add this to `DatabaseService.cs`:

```csharp
public async Task SeedDatabaseAsync()
{
    // Check if data already exists
    using var connection = CreateConnection();
    await connection.OpenAsync();

    using var command = new NpgsqlCommand("SELECT COUNT(*) FROM docks", connection);
    var count = Convert.ToInt32(await command.ExecuteScalarAsync());

    if (count > 0)
    {
        // Data already exists, no need to seed
        return;
    }

    // Seed docks
    await ExecuteNonQueryAsync(@"
        INSERT INTO docks (location, capacity) VALUES
        ('North Harbor', 5),
        ('South Harbor', 3),
        ('East Harbor', 7)
    ");

    // Seed haulers
    await ExecuteNonQueryAsync(@"
        INSERT INTO haulers (name, capacity) VALUES
        ('Oceanic Haulers', 10),
        ('Maritime Transport', 15),
        ('Sea Logistics', 8)
    ");

    // Seed ships
    await ExecuteNonQueryAsync(@"
        INSERT INTO ships (name, type, dock_id) VALUES
        ('Serenity', 'Firefly-class transport ship', 1),
        ('Rocinante', 'Corvette-class frigate', 2),
        ('Millennium Falcon', 'YT-1300 light freighter', 3)
    ");
}
```

## Step 7: Implementing Data Access Methods

Now, let's add methods to our `DatabaseService` to perform CRUD operations. Add these to `DatabaseService.cs`:

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
    command.Parameters.AddWithValue("@dockId", ship.DockId ?? DBNull.Value);

    ship.Id = Convert.ToInt32(await command.ExecuteScalarAsync());

    return ship;
}

// Update a ship
public async Task<bool> UpdateShipAsync(int id, Ship ship)
{
    using var connection = CreateConnection();
    await connection.OpenAsync();

    using var command = new NpgsqlCommand(
        @"UPDATE ships
          SET name = @name, type = @type, dock_id = @dockId
          WHERE id = @id",
        connection);

    command.Parameters.AddWithValue("@id", id);
    command.Parameters.AddWithValue("@name", ship.Name);
    command.Parameters.AddWithValue("@type", ship.Type);
    command.Parameters.AddWithValue("@dockId", ship.DockId ?? DBNull.Value);

    int rowsAffected = await command.ExecuteNonQueryAsync();

    return rowsAffected > 0;
}

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

// Get ships by dock ID
public async Task<List<Ship>> GetShipsByDockIdAsync(int dockId)
{
    var ships = new List<Ship>();

    using var connection = CreateConnection();
    await connection.OpenAsync();

    using var command = new NpgsqlCommand(
        "SELECT id, name, type, dock_id FROM ships WHERE dock_id = @dockId",
        connection);

    command.Parameters.AddWithValue("@dockId", dockId);

    using var reader = await command.ExecuteReaderAsync();

    while (await reader.ReadAsync())
    {
        ships.Add(new Ship
        {
            Id = reader.GetInt32(0),
            Name = reader.GetString(1),
            Type = reader.GetString(2),
            DockId = reader.GetInt32(3)
        });
    }

    return ships;
}

// Get all docks
public async Task<List<Dock>> GetAllDocksAsync()
{
    var docks = new List<Dock>();

    using var connection = CreateConnection();
    await connection.OpenAsync();

    using var command = new NpgsqlCommand("SELECT id, location, capacity FROM docks", connection);
    using var reader = await command.ExecuteReaderAsync();

    while (await reader.ReadAsync())
    {
        docks.Add(new Dock
        {
            Id = reader.GetInt32(0),
            Location = reader.GetString(1),
            Capacity = reader.GetInt32(2)
        });
    }

    return docks;
}

// Get dock by ID
public async Task<Dock?> GetDockByIdAsync(int id)
{
    using var connection = CreateConnection();
    await connection.OpenAsync();

    using var command = new NpgsqlCommand(
        "SELECT id, location, capacity FROM docks WHERE id = @id",
        connection);
    command.Parameters.AddWithValue("@id", id);

    using var reader = await command.ExecuteReaderAsync();

    if (await reader.ReadAsync())
    {
        return new Dock
        {
            Id = reader.GetInt32(0),
            Location = reader.GetString(1),
            Capacity = reader.GetInt32(2)
        };
    }

    return null;
}

// Get available docks (docks with available capacity)
public async Task<List<Dock>> GetAvailableDocksAsync()
{
    var docks = new List<Dock>();

    using var connection = CreateConnection();
    await connection.OpenAsync();

    using var command = new NpgsqlCommand(
        @"SELECT d.id, d.location, d.capacity, COUNT(s.id) as ship_count
          FROM docks d
          LEFT JOIN ships s ON d.id = s.dock_id
          GROUP BY d.id, d.location, d.capacity
          HAVING COUNT(s.id) < d.capacity",
        connection);

    using var reader = await command.ExecuteReaderAsync();

    while (await reader.ReadAsync())
    {
        docks.Add(new Dock
        {
            Id = reader.GetInt32(0),
            Location = reader.GetString(1),
            Capacity = reader.GetInt32(2)
        });
    }

    return docks;
}

// Get ships by type (for query string parameter example)
public async Task<List<Ship>> GetShipsByTypeAsync(string type)
{
    var ships = new List<Ship>();

    using var connection = CreateConnection();
    await connection.OpenAsync();

    using var command = new NpgsqlCommand(
        "SELECT id, name, type, dock_id FROM ships WHERE type ILIKE @type",
        connection);

    command.Parameters.AddWithValue("@type", $"%{type}%");

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
```

## Step 8: Defining API Endpoints

Now, let's define our API endpoints. Create the following files:

1. Create `Endpoints/ShipEndpoints.cs`:

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
                await db.GetAllShipsAsync());

            // GET /ships/{id} - Get a ship by ID
            app.MapGet("/ships/{id}", async (int id, DatabaseService db) =>
            {
                var ship = await db.GetShipByIdAsync(id);
                return ship != null ? Results.Ok(ship) : Results.NotFound();
            });

            // GET /ships/search - Get ships by type (query string parameter)
            app.MapGet("/ships/search", async (string? type, DatabaseService db) =>
            {
                if (string.IsNullOrEmpty(type))
                {
                    return Results.BadRequest("Type parameter is required");
                }

                var ships = await db.GetShipsByTypeAsync(type);
                return Results.Ok(ships);
            });

            // POST /ships - Create a new ship
            app.MapPost("/ships", async (Ship ship, DatabaseService db) =>
            {
                try
                {
                    var newShip = await db.CreateShipAsync(ship);
                    return Results.Created($"/ships/{newShip.Id}", newShip);
                }
                catch (Exception ex)
                {
                    return Results.Problem($"An error occurred while creating the ship: {ex.Message}");
                }
            });

            // PUT /ships/{id} - Update a ship
            app.MapPut("/ships/{id}", async (int id, Ship updatedShip, DatabaseService db) =>
            {
                try
                {
                    var success = await db.UpdateShipAsync(id, updatedShip);
                    return success ? Results.NoContent() : Results.NotFound();
                }
                catch (Exception ex)
                {
                    return Results.Problem($"An error occurred while updating the ship: {ex.Message}");
                }
            });

            // DELETE /ships/{id} - Delete a ship
            app.MapDelete("/ships/{id}", async (int id, DatabaseService db) =>
            {
                try
                {
                    var success = await db.DeleteShipAsync(id);
                    return success ? Results.NoContent() : Results.NotFound();
                }
                catch (Exception ex)
                {
                    return Results.Problem($"An error occurred while deleting the ship: {ex.Message}");
                }
            });
        }
    }
}
```

2. Create `Endpoints/DockEndpoints.cs`:

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
                await db.GetAllDocksAsync());

            // GET /docks/{id} - Get a dock by ID
            app.MapGet("/docks/{id}", async (int id, DatabaseService db) =>
            {
                var dock = await db.GetDockByIdAsync(id);
                return dock != null ? Results.Ok(dock) : Results.NotFound();
            });

            // GET /docks/{id}/ships - Get all ships at a dock
            app.MapGet("/docks/{id}/ships", async (int id, DatabaseService db) =>
            {
                var dock = await db.GetDockByIdAsync(id);
                if (dock == null)
                {
                    return Results.NotFound();
                }

                var ships = await db.GetShipsByDockIdAsync(id);
                return Results.Ok(ships);
            });

            // GET /docks/available - Get all docks with available capacity
            app.MapGet("/docks/available", async (DatabaseService db) =>
            {
                var docks = await db.GetAvailableDocksAsync();
                return Results.Ok(docks);
            });
        }
    }
}
```

## Step 9: Updating Program.cs

Now, let's update the `Program.cs` file to use our services and endpoints:

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

// Map endpoints
app.MapShipEndpoints();
app.MapDockEndpoints();

app.Run();
```

## Step 10: Running and Testing the API

Now, let's run and test our API:

1. Start the API:

```bash
dotnet run
```

2. Open Swagger at `https://localhost:7042/swagger` (or the URL shown in your terminal)
3. Test the endpoints to verify that they're working with the database

### Testing GET Requests

1. Try the `GET /ships` endpoint to get all ships
2. Try the `GET /ships/{id}` endpoint with ID 1 to get a specific ship
3. Try the `GET /ships/search?type=transport` endpoint to search for ships by type

### Testing POST Requests

1. Try the `POST /ships` endpoint to create a new ship:
   ```json
   {
     "name": "Black Pearl",
     "type": "Galleon",
     "dockId": 1
   }
   ```

2. Verify the ship was created by using the `GET /ships` endpoint

### Testing Query String Parameters

1. Try the `GET /ships/search?type=frigate` endpoint to search for ships with "frigate" in their type
2. Try the `GET /docks/available` endpoint to get docks with available capacity

## Conclusion

In this chapter, you've built a complete Harbor Master API that:
- Uses ASP.NET Core minimal API
- Connects to a PostgreSQL database using Npgsql directly (without EF Core)
- Seeds the database with initial data using SQL
- Handles GET, POST, PUT, and DELETE requests
- Processes query string parameters

This project demonstrates how to build a robust API that interacts with a database without relying on an ORM like Entity Framework Core. By using Npgsql directly, you have more control over the SQL queries and can optimize them for your specific needs.

## Next Steps

You can enhance your Harbor Master API by:
1. Adding authentication and authorization
2. Implementing logging and error handling
3. Adding validation for incoming requests
4. Creating a client application to consume the API
5. Adding more complex queries and reports