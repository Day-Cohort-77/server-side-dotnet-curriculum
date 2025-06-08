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
        ('Millennium Falcon', 'YT-1300 light freighter', 3),
        ('Black Pearl', 'Pirate galleon', 1),
        ('Nautilus', 'Submarine vessel', 2),
        ('Flying Dutchman', 'Ghost ship', 3),
        ('Enterprise', 'Constitution-class starship', 1),
        ('Voyager', 'Intrepid-class starship', 2),
        ('Defiant', 'Escort-class warship', 3),
        ('Galactica', 'Battlestar', 1),
        ('Bebop', 'Fishing trawler', 2),
        ('Normandy', 'Stealth frigate', 3),
        ('Pillar of Autumn', 'Halcyon-class cruiser', 1),
        ('Nostromo', 'Commercial towing vessel', 2),
        ('Sulaco', 'Military transport', 3),
        ('Highwind', 'Airship', 1),
        ('Argo', 'Ancient Greek galley', 2),
        ('Nebuchadnezzar', 'Hovership', 3)
    ");
}
```

This method:
1. Checks if data already exists in the docks table
2. If no data exists, it inserts sample data for docks, haulers, and ships
3. Uses the `ExecuteNonQueryAsync` helper method we created earlier to execute the SQL commands

## Updating Program.cs to Seed the Database

Now, let's update the `Program.cs` file to call our seeding method when the application starts. Add the following line of code right beneath the line where `InitializeDatabaseAsync();` is invoked.

```cs
await dbService.SeedDatabaseAsync();
```

Now when you restart the debugger to activate this code, it will perform two tasks.

1. Initialize the database
2. Add some starter data _(a.k.a. seeding)_ to the database

## Testing the Seeding

Let's run the application to test our seeding logic:

1. Restart the debugger, which will execute all of your database seeding logic
2. Go to the **SQLTools** extension panel in VSCode
3. Click **Add New Connection**
4. Enter **HarborMaster** as the connection name
5. Enter **harbormaster** as the database
6. Enter **postgres** for the username
7. Scroll down and click **Test Connection**
8. You will be asked to enter in your postgres user password
9. You should then see **Successfully connected!** message
10. Click the **Save Connection** button

You will see a new connection in your SQLTools panel. To view the data:

1. Expand the connection until you see the **Tables** item.
2. Once you expand that, you should see the docks, haulers, and ships tables.
3. Click the icon of the magnifying glass with the plus sign inside of it for each table
4. Verify that each table has data in it

![walkthrough of connecting to database with sqltools](./images/harbormaster-db-connect.gif)

If you have issues with any of these steps, reach out to your instructor ASAP.

## Conclusion

In this chapter, you've learned how to seed a PostgreSQL database with initial data using SQL queries through Npgsql. You've implemented a method to check if data already exists and to insert sample data for the Harbor Master project.

In the next chapter, we'll learn how to implement GET operations to retrieve ship data from the database.

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
        ('Millennium Falcon', 'YT-1300 light freighter', 3),
        ('Black Pearl', 'Pirate galleon', 1),
        ('Nautilus', 'Submarine vessel', 2),
        ('Flying Dutchman', 'Ghost ship', 3),
        ('Enterprise', 'Constitution-class starship', 1),
        ('Voyager', 'Intrepid-class starship', 2),
        ('Defiant', 'Escort-class warship', 3),
        ('Galactica', 'Battlestar', 1),
        ('Bebop', 'Fishing trawler', 2),
        ('Normandy', 'Stealth frigate', 1),
        ('Pillar of Autumn', 'Halcyon-class cruiser', 1),
        ('Nostromo', 'Commercial towing vessel', 2),
        ('Sulaco', 'Military transport', 3),
        ('Highwind', 'Airship', 1),
        ('Argo', 'Ancient Greek galley', 2),
        ('Nebuchadnezzar', 'Hovership', 3)
        ;
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

## Next Steps

[Continue to the next chapter: Handling GET Operations for Ships](./harbor-master-get-ships.md)