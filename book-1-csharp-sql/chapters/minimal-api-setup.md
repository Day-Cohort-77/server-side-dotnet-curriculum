# Creating a Minimal API Project

In this chapter, we'll learn how to create a minimal API project using ASP.NET Core. Minimal APIs are a simplified approach to building HTTP APIs with ASP.NET Core, requiring minimal setup code and dependencies.

## Learning Objectives

By the end of this chapter, you should be able to:
- Understand what minimal APIs are and when to use them
- Create a new minimal API project using the .NET CLI
- Configure the minimal API project
- Define endpoints using different HTTP methods
- Run and test the API using Swagger and Postman

## What are Minimal APIs?

Minimal APIs are a feature introduced in ASP.NET Core 6.0 that allows you to build HTTP APIs with minimal dependencies and setup code. They're ideal for microservices, simple HTTP APIs, and situations where you want to minimize the complexity of your application.

Key benefits of minimal APIs include:
- Less ceremony and boilerplate code
- Simplified dependency injection
- Streamlined request handling
- Easy to understand and maintain
- Fast startup time and low memory footprint

## Creating a New Minimal API Project

Let's create a new minimal API project for our Harbor Master application, which will manage ships, docks, and haulers in a harbor.

1. Open a terminal or command prompt
2. Navigate to the directory where you want to create your project
3. Run the following command:

```bash
dotnet new webapi -minimal -o HarborMaster
```

This command creates a new minimal API project with the following parameters:
- `webapi`: The project template to use
- `-minimal`: A flag that indicates we want to create a minimal API
- `-o HarborMaster`: The output directory for the project

4. Navigate to the project directory:

```bash
cd HarborMaster
```

5. Open the project in Visual Studio Code:

```bash
code .
```

## Exploring the Project Structure

Let's take a look at the files and directories that were created:

- `HarborMaster.csproj`: The project file that contains configuration information
- `Program.cs`: The main entry point for the application, where the API is defined
- `appsettings.json` and `appsettings.Development.json`: Configuration files
- `Properties/launchSettings.json`: Launch settings for the application
- `HarborMaster.http`: A file for testing HTTP requests (if using the HTTP REPL)

The most important file is `Program.cs`, which contains the minimal API definition. Let's examine it:

```csharp
var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
// Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

var summaries = new[]
{
    "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
};

app.MapGet("/weatherforecast", () =>
{
    var forecast = Enumerable.Range(1, 5).Select(index =>
        new WeatherForecast
        (
            DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
            Random.Shared.Next(-20, 55),
            summaries[Random.Shared.Next(summaries.Length)]
        ))
        .ToArray();
    return forecast;
})
.WithName("GetWeatherForecast")
.WithOpenApi();

app.Run();

record WeatherForecast(DateOnly Date, int TemperatureC, string? Summary)
{
    public int TemperatureF => 32 + (int)(TemperatureC / 0.5556);
}
```

This file sets up a minimal API with a single endpoint (`/weatherforecast`) that returns a weather forecast. Let's break it down:

1. `WebApplication.CreateBuilder(args)`: Creates a builder for configuring the application
2. `builder.Services.AddEndpointsApiExplorer()` and `builder.Services.AddSwaggerGen()`: Add services for API explorer and Swagger
3. `builder.Build()`: Builds the application
4. `app.UseSwagger()` and `app.UseSwaggerUI()`: Configure Swagger for API documentation (only in development)
5. `app.UseHttpsRedirection()`: Redirects HTTP requests to HTTPS
6. `app.MapGet("/weatherforecast", ...)`: Maps a GET request to the `/weatherforecast` endpoint
7. `app.Run()`: Runs the application

## Modifying the Project for Harbor Master

Now, let's modify the project to create our Harbor Master API. We'll start by removing the weather forecast code and adding our own endpoints.

Update the `Program.cs` file to look like this:

```csharp
var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
// Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

// Define some sample data
var ships = new List<Ship>
{
    new Ship { Id = 1, Name = "Serenity", Type = "Firefly-class transport ship" },
    new Ship { Id = 2, Name = "Rocinante", Type = "Corvette-class frigate" },
    new Ship { Id = 3, Name = "Millennium Falcon", Type = "YT-1300 light freighter" }
};

var docks = new List<Dock>
{
    new Dock { Id = 1, Location = "North Harbor", Capacity = 5 },
    new Dock { Id = 2, Location = "South Harbor", Capacity = 3 },
    new Dock { Id = 3, Location = "East Harbor", Capacity = 7 }
};

// Define API endpoints
app.MapGet("/", () => "Welcome to Harbor Master API!");

// GET /ships - Get all ships
app.MapGet("/ships", () => ships)
    .WithName("GetAllShips")
    .WithOpenApi();

// GET /ships/{id} - Get a ship by ID
app.MapGet("/ships/{id}", (int id) =>
{
    var ship = ships.FirstOrDefault(s => s.Id == id);
    return ship != null ? Results.Ok(ship) : Results.NotFound();
})
.WithName("GetShipById")
.WithOpenApi();

// GET /docks - Get all docks
app.MapGet("/docks", () => docks)
    .WithName("GetAllDocks")
    .WithOpenApi();

// GET /docks/{id} - Get a dock by ID
app.MapGet("/docks/{id}", (int id) =>
{
    var dock = docks.FirstOrDefault(d => d.Id == id);
    return dock != null ? Results.Ok(dock) : Results.NotFound();
})
.WithName("GetDockById")
.WithOpenApi();

app.Run();

// Define model classes
class Ship
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public string Type { get; set; } = string.Empty;
}

class Dock
{
    public int Id { get; set; }
    public string Location { get; set; } = string.Empty;
    public int Capacity { get; set; }
}
```

In this updated code, we've:
1. Removed the weather forecast code
2. Added sample data for ships and docks
3. Defined several API endpoints:
   - `GET /`: A simple welcome message
   - `GET /ships`: Returns all ships
   - `GET /ships/{id}`: Returns a ship by ID
   - `GET /docks`: Returns all docks
   - `GET /docks/{id}`: Returns a dock by ID
4. Defined model classes for `Ship` and `Dock`

## Running the API

Now, let's run the API to see it in action:

1. In the terminal, run the following command:

```bash
dotnet run
```

2. The API will start running, and you should see output similar to this:

```
info: Microsoft.Hosting.Lifetime[14]
      Now listening on: https://localhost:7042
info: Microsoft.Hosting.Lifetime[14]
      Now listening on: http://localhost:5042
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Development
```

3. Open a web browser and navigate to `https://localhost:7042/swagger` (or the URL shown in your terminal)
4. You should see the Swagger UI, which provides documentation for your API and allows you to test the endpoints

## Testing the API with Swagger

Swagger provides a user-friendly interface for testing your API:

1. Click on the `GET /ships` endpoint
2. Click the "Try it out" button
3. Click the "Execute" button
4. You should see a response with the list of ships

You can also test the other endpoints in a similar way.

## Adding POST, PUT, and DELETE Endpoints

Now, let's add endpoints for creating, updating, and deleting ships:

```csharp
// POST /ships - Create a new ship
app.MapPost("/ships", (Ship ship) =>
{
    // Generate a new ID (in a real app, this would be handled by the database)
    ship.Id = ships.Count > 0 ? ships.Max(s => s.Id) + 1 : 1;
    ships.Add(ship);
    return Results.Created($"/ships/{ship.Id}", ship);
})
.WithName("CreateShip")
.WithOpenApi();

// PUT /ships/{id} - Update a ship
app.MapPut("/ships/{id}", (int id, Ship updatedShip) =>
{
    var index = ships.FindIndex(s => s.Id == id);
    if (index == -1)
    {
        return Results.NotFound();
    }

    // Preserve the ID
    updatedShip.Id = id;
    ships[index] = updatedShip;
    return Results.NoContent();
})
.WithName("UpdateShip")
.WithOpenApi();

// DELETE /ships/{id} - Delete a ship
app.MapDelete("/ships/{id}", (int id) =>
{
    var index = ships.FindIndex(s => s.Id == id);
    if (index == -1)
    {
        return Results.NotFound();
    }

    ships.RemoveAt(index);
    return Results.NoContent();
})
.WithName("DeleteShip")
.WithOpenApi();
```

Add these endpoints to your `Program.cs` file, just before the `app.Run()` line.

## Testing with Postman

While Swagger is great for testing during development, you might also want to use Postman for more advanced testing:

1. Download and install Postman from [postman.com](https://www.postman.com/downloads/)
2. Open Postman and create a new request
3. Set the request method (GET, POST, PUT, DELETE) and URL (e.g., `https://localhost:7042/ships`)
4. For POST and PUT requests, set the body to JSON and provide the necessary data
5. Click "Send" to execute the request

For example, to create a new ship:
1. Set the method to POST
2. Set the URL to `https://localhost:7042/ships`
3. Set the body to JSON and enter:
```json
{
    "name": "Black Pearl",
    "type": "Galleon"
}
```
4. Click "Send"
5. You should receive a 201 Created response with the new ship data

## Organizing Your Code

As your API grows, you might want to organize your code better. Let's refactor our code to separate the model classes and endpoints:

1. Create a new folder called `Models`
2. Create a new file called `Models/Ship.cs`:

```csharp
namespace HarborMaster.Models
{
    public class Ship
    {
        public int Id { get; set; }
        public string Name { get; set; } = string.Empty;
        public string Type { get; set; } = string.Empty;
    }
}
```

3. Create a new file called `Models/Dock.cs`:

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

4. Update your `Program.cs` file to use these model classes:

```csharp
using HarborMaster.Models;

var builder = WebApplication.CreateBuilder(args);

// Rest of the code...
```

5. Remove the model class definitions from the end of the `Program.cs` file

You can also organize your endpoints by feature or resource. For example, you could create extension methods for registering endpoints:

1. Create a new folder called `Endpoints`
2. Create a new file called `Endpoints/ShipEndpoints.cs`:

```csharp
using HarborMaster.Models;

namespace HarborMaster.Endpoints
{
    public static class ShipEndpoints
    {
        public static void MapShipEndpoints(this WebApplication app, List<Ship> ships)
        {
            // GET /ships - Get all ships
            app.MapGet("/ships", () => ships)
                .WithName("GetAllShips")
                .WithOpenApi();

            // GET /ships/{id} - Get a ship by ID
            app.MapGet("/ships/{id}", (int id) =>
            {
                var ship = ships.FirstOrDefault(s => s.Id == id);
                return ship != null ? Results.Ok(ship) : Results.NotFound();
            })
            .WithName("GetShipById")
            .WithOpenApi();

            // POST /ships - Create a new ship
            app.MapPost("/ships", (Ship ship) =>
            {
                // Generate a new ID
                ship.Id = ships.Count > 0 ? ships.Max(s => s.Id) + 1 : 1;
                ships.Add(ship);
                return Results.Created($"/ships/{ship.Id}", ship);
            })
            .WithName("CreateShip")
            .WithOpenApi();

            // PUT /ships/{id} - Update a ship
            app.MapPut("/ships/{id}", (int id, Ship updatedShip) =>
            {
                var index = ships.FindIndex(s => s.Id == id);
                if (index == -1)
                {
                    return Results.NotFound();
                }

                // Preserve the ID
                updatedShip.Id = id;
                ships[index] = updatedShip;
                return Results.NoContent();
            })
            .WithName("UpdateShip")
            .WithOpenApi();

            // DELETE /ships/{id} - Delete a ship
            app.MapDelete("/ships/{id}", (int id) =>
            {
                var index = ships.FindIndex(s => s.Id == id);
                if (index == -1)
                {
                    return Results.NotFound();
                }

                ships.RemoveAt(index);
                return Results.NoContent();
            })
            .WithName("DeleteShip")
            .WithOpenApi();
        }
    }
}
```

3. Create a similar file for dock endpoints
4. Update your `Program.cs` file to use these extension methods:

```csharp
using HarborMaster.Models;
using HarborMaster.Endpoints;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
// Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

// Define some sample data
var ships = new List<Ship>
{
    new Ship { Id = 1, Name = "Serenity", Type = "Firefly-class transport ship" },
    new Ship { Id = 2, Name = "Rocinante", Type = "Corvette-class frigate" },
    new Ship { Id = 3, Name = "Millennium Falcon", Type = "YT-1300 light freighter" }
};

var docks = new List<Dock>
{
    new Dock { Id = 1, Location = "North Harbor", Capacity = 5 },
    new Dock { Id = 2, Location = "South Harbor", Capacity = 3 },
    new Dock { Id = 3, Location = "East Harbor", Capacity = 7 }
};

// Define API endpoints
app.MapGet("/", () => "Welcome to Harbor Master API!");

// Map ship endpoints
app.MapShipEndpoints(ships);

// Map dock endpoints
app.MapDockEndpoints(docks);

app.Run();
```

This approach helps keep your code organized and maintainable as your API grows.

## Conclusion

In this chapter, you've learned how to create a minimal API project using ASP.NET Core. You've defined endpoints for different HTTP methods, organized your code, and tested your API using Swagger and Postman.

In the next chapter, we'll explore how to connect our minimal API to a PostgreSQL database using Entity Framework Core.

## Practice Exercise

Enhance your Harbor Master API by:
1. Adding a new model class called `Hauler` with properties for `Id`, `Name`, and `Capacity`
2. Adding sample data for haulers
3. Creating endpoints for managing haulers (GET, POST, PUT, DELETE)
4. Testing the new endpoints using Swagger or Postman
5. Organizing the hauler endpoints using the extension method approach