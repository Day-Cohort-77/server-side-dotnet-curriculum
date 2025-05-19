# Using `DbContext`

In this chapter, we'll create the `DbContext` class for our Creek River Campground API. The `DbContext` class is a crucial component in Entity Framework Core that serves as the primary point of interaction with the database.

## What is DbContext?

`DbContext` is a class that represents a session with the database. It provides the following capabilities:

1. **Database Connection Management**: It manages the connection to the database.
2. **Entity Tracking**: It keeps track of changes made to entities during the lifetime of a context instance.
3. **CRUD Operations**: It provides methods to query and save data to the database.
4. **Model Configuration**: It allows you to configure how your entity classes map to database tables.

## Creating the DbContext Class

Let's create our `DbContext` class for the Creek River Campground API:

1. Add a file to the main directory of the project called `CreekRiverDbContext.cs`.
2. Add the following code to the file:

```csharp
using Microsoft.EntityFrameworkCore;
using CreekRiver.Models;

public class CreekRiverDbContext : DbContext
{
    public DbSet<Reservation> Reservations { get; set; }
    public DbSet<UserProfile> UserProfiles { get; set; }
    public DbSet<Campsite> Campsites { get; set; }
    public DbSet<CampsiteType> CampsiteTypes { get; set; }

    public CreekRiverDbContext(DbContextOptions<CreekRiverDbContext> context) : base(context)
    {
    }
}
```

Let's break down this code:

- **Inheritance**: Our `CreekRiverDbContext` class inherits from the `DbContext` class provided by Entity Framework Core. This gives our class all the functionality of `DbContext`.

- **DbSet Properties**: Each `DbSet<T>` property represents a table in the database. Entity Framework Core will create tables for each of these properties.

- **Constructor**: The constructor takes a `DbContextOptions<CreekRiverDbContext>` parameter and passes it to the base class constructor. This is how configuration options are passed to the `DbContext`.

## Seeding the Database

Seeding is the process of populating a database with initial data. Entity Framework Core provides a way to seed data through the `OnModelCreating` method.

Add the following method to your `CreekRiverDbContext` class:

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Seed data with campsite types
    modelBuilder.Entity<CampsiteType>().HasData(new CampsiteType[]
    {
        new CampsiteType {Id = 1, CampsiteTypeName = "Tent", FeePerNight = 15.99M, MaxReservationDays = 7},
        new CampsiteType {Id = 2, CampsiteTypeName = "RV", FeePerNight = 26.50M, MaxReservationDays = 14},
        new CampsiteType {Id = 3, CampsiteTypeName = "Primitive", FeePerNight = 10.00M, MaxReservationDays = 3},
        new CampsiteType {Id = 4, CampsiteTypeName = "Hammock", FeePerNight = 12M, MaxReservationDays = 7}
    });

    // Seed data with campsites
    modelBuilder.Entity<Campsite>().HasData(new Campsite[]
    {
        new Campsite {Id = 1, CampsiteTypeId = 1, Nickname = "Barred Owl", ImageUrl="https://tnstateparks.com/assets/images/content-images/campgrounds/249/colsp-area2-site73.jpg"},
        new Campsite {Id = 2, CampsiteTypeId = 2, Nickname = "Red Fox", ImageUrl="https://tnstateparks.com/assets/images/content-images/campgrounds/249/colsp-area2-site73.jpg"},
        new Campsite {Id = 3, CampsiteTypeId = 3, Nickname = "Wild Turkey", ImageUrl="https://tnstateparks.com/assets/images/content-images/campgrounds/249/colsp-area2-site73.jpg"},
        new Campsite {Id = 4, CampsiteTypeId = 4, Nickname = "Raccoon", ImageUrl="https://tnstateparks.com/assets/images/content-images/campgrounds/249/colsp-area2-site73.jpg"},
        new Campsite {Id = 5, CampsiteTypeId = 1, Nickname = "Gray Squirrel", ImageUrl="https://tnstateparks.com/assets/images/content-images/campgrounds/249/colsp-area2-site73.jpg"},
        new Campsite {Id = 6, CampsiteTypeId = 2, Nickname = "White-tailed Deer", ImageUrl="https://tnstateparks.com/assets/images/content-images/campgrounds/249/colsp-area2-site73.jpg"}
    });

    // Seed data with user profiles
    modelBuilder.Entity<UserProfile>().HasData(new UserProfile[]
    {
        new UserProfile {Id = 1, FirstName = "Admina", LastName = "Strator", Email = "admina@creekriver.campground"}
    });

    // Seed data with reservations
    modelBuilder.Entity<Reservation>().HasData(new Reservation[]
    {
        new Reservation {Id = 1, CampsiteId = 1, UserProfileId = 1, CheckinDate = DateTime.Parse("2023-06-10"), CheckoutDate = DateTime.Parse("2023-06-13")}
    });
}
```

This method overrides the `OnModelCreating` method from the base `DbContext` class. It uses the `ModelBuilder` parameter to configure the model and seed data.

The `HasData` method is used to seed data for each entity type. The data provided will be inserted into the database when migrations are applied.

## Understanding Object-Oriented Programming Concepts

Let's take a moment to understand some important OOP concepts used in our `DbContext` class:

1. **Inheritance**: Our `CreekRiverDbContext` class inherits from the `DbContext` class. This means it gets all the properties, methods, and behaviors of the `DbContext` class.

2. **Constructor**: The constructor is a special method that is called when an instance of a class is created. It has the same name as the class and no return type.

3. **Access Modifiers**: The `protected` keyword is an access modifier that makes the method accessible only within the class itself and by derived classes.

4. **Method Overriding**: The `override` keyword is used to modify a method that is inherited from a base class. The `OnModelCreating` method is marked as `virtual` in the base class, which allows it to be overridden.

## Conclusion

In this chapter, we've created the `DbContext` class for our Creek River Campground API. We've learned how to:

1. Create a `DbContext` class
2. Define `DbSet` properties for our entity types
3. Create instructios to seed the database with initial data
4. Configure our web API to use EF Core

In the next chapter, we'll create our first migration to create the database tables and seed the database with initial data.

Up Next: [Creating your first migration](./creek-river-migration.md)

## 🔍 Additional Materials

- [Entity Framework Core - DbContext](https://docs.microsoft.com/en-us/ef/core/dbcontext-configuration/)
- [Entity Framework Core - Seeding Data](https://docs.microsoft.com/en-us/ef/core/modeling/data-seeding)
- [C# Inheritance](https://docs.microsoft.com/en-us/dotnet/csharp/fundamentals/tutorials/inheritance)
- [C# Constructors](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/constructors)