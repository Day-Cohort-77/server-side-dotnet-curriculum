# Harbor Master: Initializing the Project and Database

In this chapter, we'll set up the Harbor Master project from scratch. We'll create a minimal API project and establish a connection to a PostgreSQL database without using Entity Framework Core.

## Learning Objectives

By the end of this chapter, you should be able to:
- Create a minimal API project in ASP.NET Core
- Set up a project structure with appropriate folders
- Connect to a PostgreSQL database using Npgsql directly
- Create a database schema with tables and relationships

## Prerequisites

Before starting this project, ensure you have:
- .NET 8 SDK installed
- PostgreSQL installed and running
- pgAdmin or another PostgreSQL client tool
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

## Step 6: Updating Program.cs

Now, let's update the `Program.cs` file to use our database service:

```csharp
using HarborMaster.Services;

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

// Initialize the database
using (var scope = app.Services.CreateScope())
{
    var dbService = scope.ServiceProvider.GetRequiredService<DatabaseService>();
    await dbService.InitializeDatabaseAsync();
}

// Define API endpoints
app.MapGet("/", () => "Welcome to Harbor Master API!");

app.Run();
```

## Step 7: Running the Project

Now, let's run the project to make sure everything is set up correctly:

1. Start the API:

```bash
dotnet run
```

2. Open Swagger at `https://localhost:7042/swagger` (or the URL shown in your terminal)
3. You should see the Swagger UI with the root endpoint

## Conclusion

In this chapter, you've set up the Harbor Master project with a minimal API and established a connection to a PostgreSQL database. You've created the necessary data models and set up the database schema.

In the next chapter, we'll learn how to seed the database with initial data.

## Next Steps

[Continue to the next chapter: Seeding the Database](./harbor-master-seeding.md)