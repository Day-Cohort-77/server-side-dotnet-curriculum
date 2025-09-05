# Create and seed your database

In this chapter, we'll learn how to use Entity Framework Core migrations to create our database schema and seed it with initial data.

## Understanding EF Core Migrations

Migrations in Entity Framework Core are a way to keep the database schema in sync with your EF Core model. They allow you to:

1. Create a database schema from your model
2. Update the schema when your model changes
3. Seed the database with initial data
4. Keep track of changes to your model over time

Each migration represents a set of changes to your model. EF Core uses these migrations to generate SQL scripts that can be applied to your database.

## Creating Your First Migration

Now that we have our models and DbContext set up, we can create our first migration:

1. Open a terminal in the main project directory.

2. Run the following command:
   ```bash
   dotnet ef migrations add InitialCreate
   ```

3. This command will create a `Migrations` folder in your project with several files:
   - `<timestamp>_InitialCreate.cs`: This file contains the migration code.
   - `<timestamp>_InitialCreate.Designer.cs`: This file contains metadata about the migration.
   - `CreekRiverDbContextModelSnapshot.cs`: This file contains a snapshot of your current model.

Let's take a look at what's in the migration file:

```csharp
public partial class InitialCreate : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.CreateTable(
            name: "CampsiteTypes",
            columns: table => new
            {
                Id = table.Column<int>(type: "integer", nullable: false)
                    .Annotation("Npgsql:ValueGenerationStrategy", NpgsqlValueGenerationStrategy.IdentityByDefaultColumn),
                CampsiteTypeName = table.Column<string>(type: "character varying(50)", maxLength: 50, nullable: false),
                MaxReservationDays = table.Column<int>(type: "integer", nullable: false),
                FeePerNight = table.Column<decimal>(type: "numeric(18,2)", nullable: false)
            },
            constraints: table =>
            {
                table.PrimaryKey("PK_CampsiteTypes", x => x.Id);
            });

        // More table creation code...

        migrationBuilder.InsertData(
            table: "CampsiteTypes",
            columns: new[] { "Id", "CampsiteTypeName", "FeePerNight", "MaxReservationDays" },
            values: new object[,]
            {
                { 1, "Tent", 15.99m, 7 },
                { 2, "RV", 26.50m, 14 },
                { 3, "Primitive", 10.00m, 3 },
                { 4, "Hammock", 12m, 7 }
            });

        // More seed data...
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        // Code to revert the migration...
    }
}
```

This file contains two methods:
- `Up()`: This method contains the code to apply the migration (create tables, add seed data, etc.).
- `Down()`: This method contains the code to revert the migration.

## Applying the Migration

Now that we've created our migration, we need to apply it to the database:

1. Run the following command:
   ```bash
   dotnet ef database update
   ```

2. This command will:
   - Create the database if it doesn't exist
   - Create the tables defined in your model
   - Seed the database with the data you provided in the `OnModelCreating` method

3. You should see output indicating that the migration was applied successfully.

## Verifying the Database

Let's verify that our database was created correctly:

1. Open your SQLTools extension.
2. Add a new connection to the `CreekRiver` database.
3. You should see the following tables:
   - `CampsiteTypes`
   - `Campsites`
   - `Reservations`
   - `UserProfiles`
   - `__EFMigrationsHistory` (this table is used by EF Core to track which migrations have been applied)

4. Open each table and verify that the seed data was created

![animated image showing the usage of the sqltools extension to check the data in each table](./images/verify-creek-river-database.gif)

## Understanding the Migration Process

Let's take a moment to understand what happened during the migration process:

1. **Migration Creation**: When we ran `dotnet ef migrations add InitialCreate`, EF Core:
   - Examined our model (the classes and their properties)
   - Compared it to the current state of the database (which was empty)
   - Generated code to create the necessary tables and relationships

2. **Migration Application**: When we ran `dotnet ef database update`, EF Core:
   - Connected to the database
   - Checked which migrations had already been applied (none in this case)
   - Applied the pending migrations in order
   - Updated the `__EFMigrationsHistory` table to record which migrations had been applied

## Adding More Migrations

As your application evolves, you'll need to make changes to your model. When you do, you can create a new migration to update the database schema:

1. Make changes to your model (add properties, add classes, etc.).
2. Run `dotnet ef migrations add <MigrationName>` to create a new migration.
3. Run `dotnet ef database update` to apply the migration.

EF Core will generate code to update the database schema to match your updated model.

## Common Migration Commands

Here are some common EF Core migration commands:

- `dotnet ef migrations add <MigrationName>`: Creates a new migration.
- `dotnet ef database update`: Applies all pending migrations.
- `dotnet ef database update <MigrationName>`: Applies migrations up to and including the specified migration.
- `dotnet ef migrations remove`: Removes the last migration (if it hasn't been applied).
- `dotnet ef migrations script`: Generates a SQL script for all migrations.
- `dotnet ef migrations script <FromMigration> <ToMigration>`: Generates a SQL script for migrations from `FromMigration` to `ToMigration`.

## Conclusion

In this chapter, we've learned how to use Entity Framework Core migrations to create our database schema and seed it with initial data. We've seen how to:

1. Create a migration
2. Apply a migration
3. Verify that the migration was applied correctly
4. Understand the migration process

Migrations are a powerful tool for managing your database schema and keeping it in sync with your application model. They allow you to make changes to your model and have those changes automatically reflected in your database.

In the next chapter, you will start the process of defining endpoints that your API will support.

Up Next: [Defining endpoint modules](./creek-river-endpoints-organization.md)

## 🔍 Additional Materials

- [Entity Framework Core - Migrations](https://docs.microsoft.com/en-us/ef/core/managing-schemas/migrations/)
- [Entity Framework Core - Seeding Data](https://docs.microsoft.com/en-us/ef/core/modeling/data-seeding)
- [Entity Framework Core - Command-Line Reference](https://docs.microsoft.com/en-us/ef/core/cli/dotnet)