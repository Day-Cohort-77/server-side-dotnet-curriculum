# Harbor Master: Seeding the Database

In this chapter, we'll learn how to seed our PostgreSQL database with initial data for the Harbor Master project. We'll use SQL queries directly through our DatabaseService instead of relying on Entity Framework Core.

## Learning Objectives

By the end of this chapter, you should be able to:
- Understand the importance of database seeding
- Implement methods to check if data already exists
- Write SQL queries to insert initial data
- Execute SQL commands using Npgsql

## Why Seed the Database?

Seeding a database means populating it with initial data that your application needs to function properly. This is useful for:

1. **Development and testing** - Having consistent test data
2. **Default data** - Providing necessary default values
3. **Demo data** - Showcasing your application's features
4. **Reference data** - Adding lookup tables and reference values

## Implementing the Seeding Method

Let's add a method to our `DatabaseService` class to seed the database with initial data. Open the `Services/DatabaseService.cs` file and add the following method:

```csharp
public async Task SeedDatabaseAsync()
{
    // Check if data already exists
    using var connection = CreateConnection();
    await connection.OpenAsync();

    // Check if docks table has data
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

This method:
1. Checks if data already exists in the docks table
2. If no data exists, it inserts sample data for docks, haulers, and ships
3. Uses the `ExecuteNonQueryAsync` helper method we created earlier to execute the SQL commands

## Updating Program.cs to Seed the Database

Now, let's update the `Program.cs` file to call our seeding method when the application starts:

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

// Initialize and seed the database
using (var scope = app.Services.CreateScope())
{
    var dbService = scope.ServiceProvider.GetRequiredService<DatabaseService>();
    await dbService.InitializeDatabaseAsync();
    await dbService.SeedDatabaseAsync();
}

// Define API endpoints
app.MapGet("/", () => "Welcome to Harbor Master API!");

app.Run();
```

We've added the call to `dbService.SeedDatabaseAsync()` after initializing the database.

## Alternative Approach: Using a SQL Script

Another approach to seeding the database is to use a SQL script. This can be useful for more complex seeding operations or when you want to keep all your database operations in SQL files.

Create a file called `seed-data.sql` in the project root:

```sql
-- Check if data exists
DO $$
BEGIN
    IF NOT EXISTS (SELECT 1 FROM docks LIMIT 1) THEN
        -- Seed docks
        INSERT INTO docks (location, capacity) VALUES
        ('North Harbor', 5),
        ('South Harbor', 3),
        ('East Harbor', 7);

        -- Seed haulers
        INSERT INTO haulers (name, capacity) VALUES
        ('Oceanic Haulers', 10),
        ('Maritime Transport', 15),
        ('Sea Logistics', 8);

        -- Seed ships
        INSERT INTO ships (name, type, dock_id) VALUES
        ('Serenity', 'Firefly-class transport ship', 1),
        ('Rocinante', 'Corvette-class frigate', 2),
        ('Millennium Falcon', 'YT-1300 light freighter', 3);
    END IF;
END $$;
```

Then, modify the `SeedDatabaseAsync` method to use this script:

```csharp
public async Task SeedDatabaseAsync()
{
    string sql = File.ReadAllText("seed-data.sql");
    await ExecuteNonQueryAsync(sql);
}
```

This approach keeps all your SQL logic in SQL files, which can be easier to manage for complex database operations.

## Testing the Seeding

Let's run the application to test our seeding logic:

1. Start the API:

```bash
dotnet run
```

2. Use pgAdmin or another PostgreSQL client to connect to your database and verify that the tables have been created and populated with data.

3. You should see:
   - 3 records in the docks table
   - 3 records in the haulers table
   - 3 records in the ships table

## Conclusion

In this chapter, you've learned how to seed a PostgreSQL database with initial data using SQL queries through Npgsql. You've implemented a method to check if data already exists and to insert sample data for the Harbor Master project.

In the next chapter, we'll learn how to implement GET operations to retrieve ship data from the database.

## Next Steps

[Continue to the next chapter: Handling GET Operations for Ships](./harbor-master-get-ships.md)