# Creating the Database

In this chapter, we'll connect our Harbor Master API to a PostgreSQL database using Entity Framework Core. This will allow us to persist our data beyond the lifetime of our application.

## Learning Objectives

By the end of this chapter, you should be able to:
- Set up Entity Framework Core for a minimal API project
- Configure a PostgreSQL database connection
- Define entity models and a database context
- Create and apply migrations
- Use Entity Framework Core to perform CRUD operations

## Setting Up Entity Framework Core

Entity Framework Core (EF Core) is an object-relational mapping (ORM) framework that allows you to work with a database using .NET objects. It eliminates the need for most of the data-access code that you typically need to write.

Let's add the necessary packages to our Harbor Master API project:

1. Open a terminal in your project directory
2. Run the following commands to install the required packages:

```bash
dotnet add package Microsoft.EntityFrameworkCore
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
```

These packages provide:
- `Microsoft.EntityFrameworkCore`: The core EF Core package
- `Microsoft.EntityFrameworkCore.Design`: Design-time components for EF Core tools
- `Npgsql.EntityFrameworkCore.PostgreSQL`: PostgreSQL provider for EF Core

## Defining Entity Models

Now, let's update our model classes to work with EF Core. We'll need to make a few changes to our existing `Ship` and `Dock` classes.

1. Update the `Models/Ship.cs` file:

```csharp
namespace HarborMaster.Models
{
    public class Ship
    {
        public int Id { get; set; }
        public string Name { get; set; } = string.Empty;
        public string Type { get; set; } = string.Empty;

        // Navigation property for the relationship with Dock
        public int? DockId { get; set; }
        public Dock? Dock { get; set; }
    }
}
```

2. Update the `Models/Dock.cs` file:

```csharp
namespace HarborMaster.Models
{
    public class Dock
    {
        public int Id { get; set; }
        public string Location { get; set; } = string.Empty;
        public int Capacity { get; set; }

        // Navigation property for the relationship with Ships
        public List<Ship> Ships { get; set; } = new List<Ship>();
    }
}
```

3. Create a new model class for haulers in `Models/Hauler.cs`:

```csharp
namespace HarborMaster.Models
{
    public class Hauler
    {
        public int Id { get; set; }
        public string Name { get; set; } = string.Empty;
        public int Capacity { get; set; }

        // Navigation property for the relationship with Ships
        public List<Ship> Ships { get; set; } = new List<Ship>();
    }
}
```

4. Update the `Ship` class to include a relationship with haulers:

```csharp
namespace HarborMaster.Models
{
    public class Ship
    {
        public int Id { get; set; }
        public string Name { get; set; } = string.Empty;
        public string Type { get; set; } = string.Empty;

        // Navigation property for the relationship with Dock
        public int? DockId { get; set; }
        public Dock? Dock { get; set; }

        // Navigation property for the relationship with Hauler
        public int? HaulerId { get; set; }
        public Hauler? Hauler { get; set; }
    }
}
```

## Creating a Database Context

The database context is the main class that coordinates Entity Framework functionality for a data model. It represents a session with the database and can be used to query and save instances of your entities.

1. Create a new folder called `Data`
2. Create a new file called `Data/HarborContext.cs`:

```csharp
using Microsoft.EntityFrameworkCore;
using HarborMaster.Models;

namespace HarborMaster.Data
{
    public class HarborContext : DbContext
    {
        public HarborContext(DbContextOptions<HarborContext> options)
            : base(options)
        {
        }

        public DbSet<Ship> Ships { get; set; }
        public DbSet<Dock> Docks { get; set; }
        public DbSet<Hauler> Haulers { get; set; }

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            // Configure the Ship entity
            modelBuilder.Entity<Ship>()
                .HasOne(s => s.Dock)
                .WithMany(d => d.Ships)
                .HasForeignKey(s => s.DockId)
                .OnDelete(DeleteBehavior.SetNull);

            modelBuilder.Entity<Ship>()
                .HasOne(s => s.Hauler)
                .WithMany(h => h.Ships)
                .HasForeignKey(s => s.HaulerId)
                .OnDelete(DeleteBehavior.SetNull);

            // Seed data
            modelBuilder.Entity<Dock>().HasData(
                new Dock { Id = 1, Location = "North Harbor", Capacity = 5 },
                new Dock { Id = 2, Location = "South Harbor", Capacity = 3 },
                new Dock { Id = 3, Location = "East Harbor", Capacity = 7 }
            );

            modelBuilder.Entity<Hauler>().HasData(
                new Hauler { Id = 1, Name = "Oceanic Haulers", Capacity = 10 },
                new Hauler { Id = 2, Name = "Maritime Transport", Capacity = 15 },
                new Hauler { Id = 3, Name = "Sea Logistics", Capacity = 8 }
            );

            modelBuilder.Entity<Ship>().HasData(
                new Ship { Id = 1, Name = "Serenity", Type = "Firefly-class transport ship", DockId = 1 },
                new Ship { Id = 2, Name = "Rocinante", Type = "Corvette-class frigate", DockId = 2 },
                new Ship { Id = 3, Name = "Millennium Falcon", Type = "YT-1300 light freighter", DockId = 3 }
            );
        }
    }
}
```

In this context class:
- We define `DbSet` properties for each entity type
- We configure the relationships between entities in the `OnModelCreating` method
- We seed the database with initial data

## Configuring the Database Connection

Now, let's configure the database connection in our `Program.cs` file:

1. Update the `appsettings.json` file to include the connection string:

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

Replace `your_password` with the password you set for your PostgreSQL user.

2. Update the `Program.cs` file to configure the database context:

```csharp
using Microsoft.EntityFrameworkCore;
using HarborMaster.Data;
using HarborMaster.Models;
using HarborMaster.Endpoints;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
// Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Add database context
builder.Services.AddDbContext<HarborContext>(options =>
    options.UseNpgsql(builder.Configuration.GetConnectionString("DefaultConnection")));

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

// Define API endpoints
app.MapGet("/", () => "Welcome to Harbor Master API!");

// Map endpoints
app.MapShipEndpoints();
app.MapDockEndpoints();
app.MapHaulerEndpoints();

app.Run();
```

## Creating and Applying Migrations

Migrations are a way to keep the database schema in sync with the EF Core model. Let's create and apply migrations:

1. Install the EF Core CLI tools (if you haven't already):

```bash
dotnet tool install --global dotnet-ef
```

2. Create the initial migration:

```bash
dotnet ef migrations add InitialCreate
```

This command creates a `Migrations` folder with files that define the database schema.

3. Apply the migration to create the database:

```bash
dotnet ef database update
```

This command creates the database and tables based on your entity models.

## Updating the Endpoints

Now, let's update our endpoints to use the database context instead of in-memory lists. We'll need to modify our endpoint extension methods.

1. Update the `Endpoints/ShipEndpoints.cs` file:

```csharp
using Microsoft.EntityFrameworkCore;
using HarborMaster.Data;
using HarborMaster.Models;

namespace HarborMaster.Endpoints
{
    public static class ShipEndpoints
    {
        public static void MapShipEndpoints(this WebApplication app)
        {
            // GET /ships - Get all ships
            app.MapGet("/ships", async (HarborContext db) =>
                await db.Ships.ToListAsync())
                .WithName("GetAllShips");

            // GET /ships/{id} - Get a ship by ID
            app.MapGet("/ships/{id}", async (int id, HarborContext db) =>
            {
                var ship = await db.Ships
                    .Include(s => s.Dock)
                    .Include(s => s.Hauler)
                    .FirstOrDefaultAsync(s => s.Id == id);

                return ship != null ? Results.Ok(ship) : Results.NotFound();
            })
            .WithName("GetShipById");

            // POST /ships - Create a new ship
            app.MapPost("/ships", async (Ship ship, HarborContext db) =>
            {
                db.Ships.Add(ship);
                await db.SaveChangesAsync();

                return Results.Created($"/ships/{ship.Id}", ship);
            })
            .WithName("CreateShip");

            // PUT /ships/{id} - Update a ship
            app.MapPut("/ships/{id}", async (int id, Ship updatedShip, HarborContext db) =>
            {
                var ship = await db.Ships.FindAsync(id);

                if (ship == null)
                {
                    return Results.NotFound();
                }

                ship.Name = updatedShip.Name;
                ship.Type = updatedShip.Type;
                ship.DockId = updatedShip.DockId;
                ship.HaulerId = updatedShip.HaulerId;

                await db.SaveChangesAsync();

                return Results.NoContent();
            });

            // DELETE /ships/{id} - Delete a ship
            app.MapDelete("/ships/{id}", async (int id, HarborContext db) =>
            {
                var ship = await db.Ships.FindAsync(id);

                if (ship == null)
                {
                    return Results.NotFound();
                }

                db.Ships.Remove(ship);
                await db.SaveChangesAsync();

                return Results.NoContent();
            });
        }
    }
}
```

2. Create similar files for `DockEndpoints.cs` and `HaulerEndpoints.cs`

## Testing the API with the Database

Now, let's run the API and test it with the database:

1. Start the API:

```bash
dotnet run
```

2. Open Swagger at `https://localhost:7042/swagger` (or the URL shown in your terminal)
3. Test the endpoints to verify that they're working with the database

## Handling Database Errors

When working with a database, it's important to handle errors properly. Let's update our endpoints to include error handling:

```csharp
// POST /ships - Create a new ship
app.MapPost("/ships", async (Ship ship, HarborContext db) =>
{
    try
    {
        db.Ships.Add(ship);
        await db.SaveChangesAsync();

        return Results.Created($"/ships/{ship.Id}", ship);
    }
    catch (Exception ex)
    {
        return Results.Problem($"An error occurred while creating the ship: {ex.Message}");
    }
});
```

Add similar error handling to the other endpoints.

## Adding More Complex Queries

Let's add some more complex queries to our API:

1. Get ships by dock:

```csharp
// GET /docks/{id}/ships - Get all ships at a dock
app.MapGet("/docks/{id}/ships", async (int id, HarborContext db) =>
{
    var dock = await db.Docks
        .Include(d => d.Ships)
        .FirstOrDefaultAsync(d => d.Id == id);

    if (dock == null)
    {
        return Results.NotFound();
    }

    return Results.Ok(dock.Ships);
});
```

2. Get ships by hauler:

```csharp
// GET /haulers/{id}/ships - Get all ships for a hauler
app.MapGet("/haulers/{id}/ships", async (int id, HarborContext db) =>
{
    var hauler = await db.Haulers
        .Include(h => h.Ships)
        .FirstOrDefaultAsync(h => h.Id == id);

    if (hauler == null)
    {
        return Results.NotFound();
    }

    return Results.Ok(hauler.Ships);
});
```

3. Get available docks (docks with available capacity):

```csharp
// GET /docks/available - Get all docks with available capacity
app.MapGet("/docks/available", async (HarborContext db) =>
{
    var docks = await db.Docks
        .Include(d => d.Ships)
        .Where(d => d.Ships.Count < d.Capacity)
        .ToListAsync();

    return Results.Ok(docks);
});
```

## Conclusion

In this chapter, you've learned how to connect your minimal API to a PostgreSQL database using Entity Framework Core. You've defined entity models, created a database context, applied migrations, and updated your endpoints to use the database.

In the next chapter, we'll explore the API in more depth and learn how to handle more complex scenarios.

## Practice Exercise

Enhance your Harbor Master API by:
1. Adding a new entity called `Cargo` with properties for `Id`, `Description`, `Weight`, and a relationship with `Ship`
2. Creating migrations and updating the database
3. Adding endpoints for managing cargo (GET, POST, PUT, DELETE)
4. Adding a query to get all cargo for a specific ship
5. Adding a query to get all ships with cargo above a certain weight