# Creating the TinyTreatsDbContext

In this chapter, we'll create the `TinyTreatsDbContext` class for our bakery management system. The database context is a crucial component in Entity Framework Core that manages the connection to the database and provides access to our data models.

## Understanding DbContext in Entity Framework Core

The `DbContext` class in Entity Framework Core serves several important purposes:

1. **Database Connection**: It manages the connection to the database
2. **Entity Mapping**: It maps C# classes to database tables
3. **Change Tracking**: It tracks changes to entities for saving to the database
4. **Query Translation**: It translates LINQ queries to SQL
5. **Data Seeding**: It provides a way to seed initial data

For our TinyTreats application, we'll create a custom `DbContext` that inherits from `IdentityDbContext` to support ASP.NET Core Identity for authentication and authorization.

## Creating the TinyTreatsDbContext Class

Let's create our database context class:

```csharp
// Data/TinyTreatsDbContext.cs
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Identity.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore;
using TinyTreats.Models;

namespace TinyTreats.Data;

public class TinyTreatsDbContext : IdentityDbContext<IdentityUser>
{
    // DbSet properties for our models
    public DbSet<UserProfile> UserProfiles { get; set; }
    public DbSet<Product> Products { get; set; }
    public DbSet<Order> Orders { get; set; }
    public DbSet<OrderItem> OrderItems { get; set; }

    public TinyTreatsDbContext(DbContextOptions<TinyTreatsDbContext> options) : base(options) { }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        // Configure entity relationships
        ConfigureRelationships(modelBuilder);

        // Seed data
        SeedData(modelBuilder);
    }

    private void ConfigureRelationships(ModelBuilder modelBuilder)
    {
        // Configure UserProfile to IdentityUser relationship (one-to-one)
        modelBuilder.Entity<UserProfile>()
            .HasOne(up => up.IdentityUser)
            .WithOne()
            .HasForeignKey<UserProfile>(up => up.IdentityUserId);

        // Configure Order to UserProfile relationship (many-to-one)
        modelBuilder.Entity<Order>()
            .HasOne(o => o.UserProfile)
            .WithMany(up => up.Orders)
            .HasForeignKey(o => o.UserProfileId)
            .OnDelete(DeleteBehavior.Restrict);

        // Configure OrderItem relationships
        modelBuilder.Entity<OrderItem>()
            .HasOne(oi => oi.Order)
            .WithMany(o => o.OrderItems)
            .HasForeignKey(oi => oi.OrderId)
            .OnDelete(DeleteBehavior.Cascade);

        modelBuilder.Entity<OrderItem>()
            .HasOne(oi => oi.Product)
            .WithMany(p => p.OrderItems)
            .HasForeignKey(oi => oi.ProductId)
            .OnDelete(DeleteBehavior.Restrict);
    }

    private void SeedData(ModelBuilder modelBuilder)
    {
        // Seed roles
        SeedRoles(modelBuilder);

        // Seed users
        SeedUsers(modelBuilder);

        // Seed user profiles
        SeedUserProfiles(modelBuilder);

        // Assign users to roles
        SeedUserRoles(modelBuilder);

        // Seed products
        SeedProducts(modelBuilder);
    }

    private void SeedRoles(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<IdentityRole>().HasData(
            new IdentityRole
            {
                Id = "fab4fac1-c546-41de-aebc-a14da6895711",
                Name = "Admin",
                NormalizedName = "ADMIN"
            },
            new IdentityRole
            {
                Id = "c7b013f0-5201-4317-abd8-c211f91b7330",
                Name = "Baker",
                NormalizedName = "BAKER"
            },
            new IdentityRole
            {
                Id = "2c5e174e-3b0e-446f-86af-483d56fd7210",
                Name = "Customer",
                NormalizedName = "CUSTOMER"
            }
        );
    }

    private void SeedUsers(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<IdentityUser>().HasData(
            // Admin user
            new IdentityUser
            {
                Id = "dbc40bc6-0829-4ac5-a3ed-180f5e916a5f",
                UserName = "admin@tinytreats.com",
                Email = "admin@tinytreats.com",
                NormalizedEmail = "ADMIN@TINYTREATS.COM",
                NormalizedUserName = "ADMIN@TINYTREATS.COM",
                EmailConfirmed = true,
                // This hashes the password "Admin123!"
                PasswordHash = new PasswordHasher<IdentityUser>().HashPassword(null, "Admin123!")
            },

            // Baker users
            new IdentityUser
            {
                Id = "e2cfe4e6-5437-4efb-9a66-8d1371796bda",
                UserName = "baker1@tinytreats.com",
                Email = "baker1@tinytreats.com",
                NormalizedEmail = "BAKER1@TINYTREATS.COM",
                NormalizedUserName = "BAKER1@TINYTREATS.COM",
                EmailConfirmed = true,
                PasswordHash = new PasswordHasher<IdentityUser>().HashPassword(null, "Baker123!")
            },
            new IdentityUser
            {
                Id = "a1ffd800-9189-4a69-a24a-9b8c094f12a5",
                UserName = "baker2@tinytreats.com",
                Email = "baker2@tinytreats.com",
                NormalizedEmail = "BAKER2@TINYTREATS.COM",
                NormalizedUserName = "BAKER2@TINYTREATS.COM",
                EmailConfirmed = true,
                PasswordHash = new PasswordHasher<IdentityUser>().HashPassword(null, "Baker123!")
            },

            // Customer users
            new IdentityUser
            {
                Id = "b9c6f5e4-d4d5-4a16-9551-b7e5e859c35a",
                UserName = "customer1@example.com",
                Email = "customer1@example.com",
                NormalizedEmail = "CUSTOMER1@EXAMPLE.COM",
                NormalizedUserName = "CUSTOMER1@EXAMPLE.COM",
                EmailConfirmed = true,
                PasswordHash = new PasswordHasher<IdentityUser>().HashPassword(null, "Customer123!")
            },
            new IdentityUser
            {
                Id = "c9d6f5e4-d4d5-4a16-9551-b7e5e859c35b",
                UserName = "customer2@example.com",
                Email = "customer2@example.com",
                NormalizedEmail = "CUSTOMER2@EXAMPLE.COM",
                NormalizedUserName = "CUSTOMER2@EXAMPLE.COM",
                EmailConfirmed = true,
                PasswordHash = new PasswordHasher<IdentityUser>().HashPassword(null, "Customer123!")
            }
        );
    }

    private void SeedUserProfiles(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<UserProfile>().HasData(
            // Admin profile
            new UserProfile
            {
                Id = 1,
                IdentityUserId = "dbc40bc6-0829-4ac5-a3ed-180f5e916a5f",
                FirstName = "Admin",
                LastName = "User",
                Address = "123 Bakery Lane"
            },

            // Baker profiles
            new UserProfile
            {
                Id = 2,
                IdentityUserId = "e2cfe4e6-5437-4efb-9a66-8d1371796bda",
                FirstName = "Jane",
                LastName = "Baker",
                Address = "456 Pastry Ave"
            },
            new UserProfile
            {
                Id = 3,
                IdentityUserId = "a1ffd800-9189-4a69-a24a-9b8c094f12a5",
                FirstName = "John",
                LastName = "Dough",
                Address = "789 Cupcake Blvd"
            },

            // Customer profiles
            new UserProfile
            {
                Id = 4,
                IdentityUserId = "b9c6f5e4-d4d5-4a16-9551-b7e5e859c35a",
                FirstName = "Alice",
                LastName = "Johnson",
                Address = "101 Sweet St"
            },
            new UserProfile
            {
                Id = 5,
                IdentityUserId = "c9d6f5e4-d4d5-4a16-9551-b7e5e859c35b",
                FirstName = "Bob",
                LastName = "Smith",
                Address = "202 Dessert Dr"
            }
        );
    }

    private void SeedUserRoles(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<IdentityUserRole<string>>().HasData(
            // Admin role assignment
            new IdentityUserRole<string>
            {
                RoleId = "fab4fac1-c546-41de-aebc-a14da6895711",
                UserId = "dbc40bc6-0829-4ac5-a3ed-180f5e916a5f"
            },

            // Baker role assignments
            new IdentityUserRole<string>
            {
                RoleId = "c7b013f0-5201-4317-abd8-c211f91b7330",
                UserId = "e2cfe4e6-5437-4efb-9a66-8d1371796bda"
            },
            new IdentityUserRole<string>
            {
                RoleId = "c7b013f0-5201-4317-abd8-c211f91b7330",
                UserId = "a1ffd800-9189-4a69-a24a-9b8c094f12a5"
            },

            // Customer role assignments
            new IdentityUserRole<string>
            {
                RoleId = "2c5e174e-3b0e-446f-86af-483d56fd7210",
                UserId = "b9c6f5e4-d4d5-4a16-9551-b7e5e859c35a"
            },
            new IdentityUserRole<string>
            {
                RoleId = "2c5e174e-3b0e-446f-86af-483d56fd7210",
                UserId = "c9d6f5e4-d4d5-4a16-9551-b7e5e859c35b"
            }
        );
    }

    private void SeedProducts(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Product>().HasData(
            new Product
            {
                Id = 1,
                Name = "Chocolate Chip Cookie",
                Description = "Classic chocolate chip cookie with a soft center and crisp edges",
                Price = 2.50m,
                IsAvailable = true,
                ImageUrl = "/images/chocolate-chip-cookie.jpg"
            },
            new Product
            {
                Id = 2,
                Name = "Vanilla Cupcake",
                Description = "Light and fluffy vanilla cupcake with buttercream frosting",
                Price = 3.75m,
                IsAvailable = true,
                ImageUrl = "/images/vanilla-cupcake.jpg"
            },
            new Product
            {
                Id = 3,
                Name = "Blueberry Muffin",
                Description = "Moist muffin packed with fresh blueberries",
                Price = 3.25m,
                IsAvailable = true,
                ImageUrl = "/images/blueberry-muffin.jpg"
            },
            new Product
            {
                Id = 4,
                Name = "Cinnamon Roll",
                Description = "Soft roll with cinnamon swirl and cream cheese frosting",
                Price = 4.50m,
                IsAvailable = true,
                ImageUrl = "/images/cinnamon-roll.jpg"
            },
            new Product
            {
                Id = 5,
                Name = "Chocolate Brownie",
                Description = "Rich and fudgy chocolate brownie",
                Price = 3.00m,
                IsAvailable = true,
                ImageUrl = "/images/chocolate-brownie.jpg"
            }
        );
    }
}
```

## Understanding the TinyTreatsDbContext Implementation

Let's break down the key components of our `TinyTreatsDbContext` class:

### DbSet Properties

The `DbSet<T>` properties represent the database tables for our models:

```csharp
public DbSet<UserProfile> UserProfiles { get; set; }
public DbSet<Product> Products { get; set; }
public DbSet<Order> Orders { get; set; }
public DbSet<OrderItem> OrderItems { get; set; }
```

These properties allow us to query and manipulate data in the corresponding tables.

### Constructor

The constructor accepts `DbContextOptions<TinyTreatsDbContext>` and passes it to the base class:

```csharp
public TinyTreatsDbContext(DbContextOptions<TinyTreatsDbContext> options) : base(options) { }
```

This allows the dependency injection system to provide the configuration options when creating an instance of the context.

### OnModelCreating Method

The `OnModelCreating` method is called when the model for a derived context is being created:

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    base.OnModelCreating(modelBuilder);

    // Configure entity relationships
    ConfigureRelationships(modelBuilder);

    // Seed data
    SeedData(modelBuilder);
}
```

We first call the base implementation to set up the Identity tables, then configure our entity relationships and seed data.

### Configuring Relationships

The `ConfigureRelationships` method configures the relationships between our entities:

```csharp
private void ConfigureRelationships(ModelBuilder modelBuilder)
{
    // Configure UserProfile to IdentityUser relationship (one-to-one)
    modelBuilder.Entity<UserProfile>()
        .HasOne(up => up.IdentityUser)
        .WithOne()
        .HasForeignKey<UserProfile>(up => up.IdentityUserId);

    // Configure Order to UserProfile relationship (many-to-one)
    modelBuilder.Entity<Order>()
        .HasOne(o => o.UserProfile)
        .WithMany(up => up.Orders)
        .HasForeignKey(o => o.UserProfileId)
        .OnDelete(DeleteBehavior.Restrict);

    // Configure OrderItem relationships
    modelBuilder.Entity<OrderItem>()
        .HasOne(oi => oi.Order)
        .WithMany(o => o.OrderItems)
        .HasForeignKey(oi => oi.OrderId)
        .OnDelete(DeleteBehavior.Cascade);

    modelBuilder.Entity<OrderItem>()
        .HasOne(oi => oi.Product)
        .WithMany(p => p.OrderItems)
        .HasForeignKey(oi => oi.ProductId)
        .OnDelete(DeleteBehavior.Restrict);
}
```

This method configures:
- A one-to-one relationship between `UserProfile` and `IdentityUser`
- A many-to-one relationship between `Order` and `UserProfile`
- A many-to-one relationship between `OrderItem` and `Order` (with cascade delete)
- A many-to-one relationship between `OrderItem` and `Product` (with restrict delete)

### Seeding Data

The `SeedData` method and its helper methods seed initial data for our application:

```csharp
private void SeedData(ModelBuilder modelBuilder)
{
    // Seed roles
    SeedRoles(modelBuilder);

    // Seed users
    SeedUsers(modelBuilder);

    // Seed user profiles
    SeedUserProfiles(modelBuilder);

    // Assign users to roles
    SeedUserRoles(modelBuilder);

    // Seed products
    SeedProducts(modelBuilder);
}
```

This includes:
- Roles (Admin, Baker, Customer)
- Users (admin, bakers, customers)
- User profiles
- User role assignments
- Products

## Delete Behaviors

Note the different delete behaviors we've configured:

- `DeleteBehavior.Cascade`: When an order is deleted, all its order items are also deleted
- `DeleteBehavior.Restrict`: Prevents deletion of a user profile if they have orders, and prevents deletion of a product if it's referenced by order items

These behaviors help maintain data integrity in our database.

## Summary

In this chapter, we've created the `TinyTreatsDbContext` class that:

1. Inherits from `IdentityDbContext` to support authentication
2. Defines `DbSet<T>` properties for our models
3. Configures relationships between entities
4. Seeds initial data for roles, users, user profiles, and products

The database context is a crucial component that bridges our C# models with the database. It provides a clean, object-oriented way to interact with our data.

In the next chapter, we'll implement the authentication endpoints for our TinyTreats application.

[Next: Configure the program and create database](./tinytreats-program.md)