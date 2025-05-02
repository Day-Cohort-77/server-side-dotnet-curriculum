# Jewelry Database

In this chapter, we'll focus on setting up and managing the database for our Jewelry Junction API. We'll explore how to create a more complex database schema, implement relationships between tables, and use Entity Framework Core to interact with the database.

## Learning Objectives

By the end of this chapter, you should be able to:
- Design a more complex database schema with relationships
- Implement database migrations with Entity Framework Core
- Seed the database with initial data
- Query related data using Entity Framework Core
- Implement data validation and constraints

## Reviewing the Database Schema

Let's review the database schema we defined in the previous chapter:

- **Products**: The jewelry items for sale
- **Metals**: The materials used to make the jewelry
- **Gemstones**: The precious stones used in the jewelry
- **ProductGemstones**: A join table for the many-to-many relationship between Products and Gemstones
- **Orders**: Customer purchases of products
- **OrderItems**: Items within an order
- **Customers**: People who place orders

This schema represents a typical e-commerce database with relationships between entities. Let's enhance it by adding some additional features.

## Enhancing the Database Schema

Let's enhance our database schema by adding a few more entities and relationships:

1. **Categories**: To categorize products (e.g., Rings, Necklaces, Bracelets)
2. **Discounts**: To apply discounts to products
3. **Reviews**: To allow customers to review products

Create the following model classes:

1. `Models/Category.cs`:

```csharp
namespace JewelryJunction.Models
{
    public class Category
    {
        public int Id { get; set; }
        public string Name { get; set; } = string.Empty;
        public string Description { get; set; } = string.Empty;

        // Relationships
        public List<Product> Products { get; set; } = new List<Product>();
    }
}
```

2. `Models/Discount.cs`:

```csharp
namespace JewelryJunction.Models
{
    public class Discount
    {
        public int Id { get; set; }
        public string Name { get; set; } = string.Empty;
        public string Description { get; set; } = string.Empty;
        public decimal DiscountPercent { get; set; }
        public DateTime StartDate { get; set; }
        public DateTime EndDate { get; set; }
        public bool IsActive { get; set; }

        // Relationships
        public List<Product> Products { get; set; } = new List<Product>();
    }
}
```

3. `Models/Review.cs`:

```csharp
namespace JewelryJunction.Models
{
    public class Review
    {
        public int Id { get; set; }
        public int Rating { get; set; } // 1-5 stars
        public string Comment { get; set; } = string.Empty;
        public DateTime ReviewDate { get; set; }

        // Relationships
        public int ProductId { get; set; }
        public Product? Product { get; set; }
        public int CustomerId { get; set; }
        public Customer? Customer { get; set; }
    }
}
```

Now, let's update the `Product` class to include these new relationships:

```csharp
namespace JewelryJunction.Models
{
    public class Product
    {
        public int Id { get; set; }
        public string Name { get; set; } = string.Empty;
        public string Description { get; set; } = string.Empty;
        public decimal Price { get; set; }
        public int StockQuantity { get; set; }
        public string Type { get; set; } = string.Empty; // Ring, Necklace, Bracelet, etc.

        // Relationships
        public int MetalId { get; set; }
        public Metal? Metal { get; set; }
        public List<ProductGemstone> ProductGemstones { get; set; } = new List<ProductGemstone>();
        public int CategoryId { get; set; }
        public Category? Category { get; set; }
        public int? DiscountId { get; set; }
        public Discount? Discount { get; set; }
        public List<Review> Reviews { get; set; } = new List<Review>();
    }
}
```

## Updating the Database Context

Now, let's update our database context to include these new entities:

```csharp
using Microsoft.EntityFrameworkCore;
using JewelryJunction.Models;

namespace JewelryJunction.Data
{
    public class JewelryContext : DbContext
    {
        public JewelryContext(DbContextOptions<JewelryContext> options)
            : base(options)
        {
        }

        public DbSet<Product> Products { get; set; }
        public DbSet<Metal> Metals { get; set; }
        public DbSet<Gemstone> Gemstones { get; set; }
        public DbSet<ProductGemstone> ProductGemstones { get; set; }
        public DbSet<Order> Orders { get; set; }
        public DbSet<OrderItem> OrderItems { get; set; }
        public DbSet<Customer> Customers { get; set; }
        public DbSet<Category> Categories { get; set; }
        public DbSet<Discount> Discounts { get; set; }
        public DbSet<Review> Reviews { get; set; }

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            // Configure relationships
            modelBuilder.Entity<Product>()
                .HasOne(p => p.Metal)
                .WithMany(m => m.Products)
                .HasForeignKey(p => p.MetalId);

            modelBuilder.Entity<ProductGemstone>()
                .HasOne(pg => pg.Product)
                .WithMany(p => p.ProductGemstones)
                .HasForeignKey(pg => pg.ProductId);

            modelBuilder.Entity<ProductGemstone>()
                .HasOne(pg => pg.Gemstone)
                .WithMany(g => g.ProductGemstones)
                .HasForeignKey(pg => pg.GemstoneId);

            modelBuilder.Entity<Order>()
                .HasOne(o => o.Customer)
                .WithMany(c => c.Orders)
                .HasForeignKey(o => o.CustomerId);

            modelBuilder.Entity<OrderItem>()
                .HasOne(oi => oi.Order)
                .WithMany(o => o.OrderItems)
                .HasForeignKey(oi => oi.OrderId);

            modelBuilder.Entity<OrderItem>()
                .HasOne(oi => oi.Product)
                .WithMany()
                .HasForeignKey(oi => oi.ProductId);

            modelBuilder.Entity<Product>()
                .HasOne(p => p.Category)
                .WithMany(c => c.Products)
                .HasForeignKey(p => p.CategoryId);

            modelBuilder.Entity<Product>()
                .HasOne(p => p.Discount)
                .WithMany(d => d.Products)
                .HasForeignKey(p => p.DiscountId);

            modelBuilder.Entity<Review>()
                .HasOne(r => r.Product)
                .WithMany(p => p.Reviews)
                .HasForeignKey(r => r.ProductId);

            modelBuilder.Entity<Review>()
                .HasOne(r => r.Customer)
                .WithMany()
                .HasForeignKey(r => r.CustomerId);

            // Seed data
            SeedData(modelBuilder);
        }

        private void SeedData(ModelBuilder modelBuilder)
        {
            // Seed Categories
            modelBuilder.Entity<Category>().HasData(
                new Category { Id = 1, Name = "Rings", Description = "Finger jewelry" },
                new Category { Id = 2, Name = "Necklaces", Description = "Neck jewelry" },
                new Category { Id = 3, Name = "Bracelets", Description = "Wrist jewelry" },
                new Category { Id = 4, Name = "Earrings", Description = "Ear jewelry" }
            );

            // Seed Metals
            modelBuilder.Entity<Metal>().HasData(
                new Metal { Id = 1, Name = "Gold", PricePerGram = 60.00m },
                new Metal { Id = 2, Name = "Silver", PricePerGram = 0.85m },
                new Metal { Id = 3, Name = "Platinum", PricePerGram = 30.00m }
            );

            // Seed Gemstones
            modelBuilder.Entity<Gemstone>().HasData(
                new Gemstone { Id = 1, Name = "Diamond", PricePerCarat = 1500.00m },
                new Gemstone { Id = 2, Name = "Ruby", PricePerCarat = 1000.00m },
                new Gemstone { Id = 3, Name = "Emerald", PricePerCarat = 800.00m },
                new Gemstone { Id = 4, Name = "Sapphire", PricePerCarat = 1200.00m }
            );

            // Seed Discounts
            modelBuilder.Entity<Discount>().HasData(
                new Discount
                {
                    Id = 1,
                    Name = "Summer Sale",
                    Description = "Summer discount on selected items",
                    DiscountPercent = 10.00m,
                    StartDate = new DateTime(2025, 6, 1),
                    EndDate = new DateTime(2025, 8, 31),
                    IsActive = true
                },
                new Discount
                {
                    Id = 2,
                    Name = "Clearance",
                    Description = "Clearance discount on old inventory",
                    DiscountPercent = 25.00m,
                    StartDate = new DateTime(2025, 5, 1),
                    EndDate = new DateTime(2025, 5, 31),
                    IsActive = true
                }
            );

            // Seed Products
            modelBuilder.Entity<Product>().HasData(
                new Product
                {
                    Id = 1,
                    Name = "Diamond Ring",
                    Description = "A beautiful diamond ring",
                    Price = 1000.00m,
                    StockQuantity = 10,
                    Type = "Ring",
                    MetalId = 1,
                    CategoryId = 1,
                    DiscountId = null
                },
                new Product
                {
                    Id = 2,
                    Name = "Silver Necklace",
                    Description = "An elegant silver necklace",
                    Price = 150.00m,
                    StockQuantity = 15,
                    Type = "Necklace",
                    MetalId = 2,
                    CategoryId = 2,
                    DiscountId = 1
                },
                new Product
                {
                    Id = 3,
                    Name = "Platinum Bracelet",
                    Description = "A luxurious platinum bracelet",
                    Price = 800.00m,
                    StockQuantity = 5,
                    Type = "Bracelet",
                    MetalId = 3,
                    CategoryId = 3,
                    DiscountId = null
                },
                new Product
                {
                    Id = 4,
                    Name = "Gold Earrings",
                    Description = "Elegant gold earrings",
                    Price = 350.00m,
                    StockQuantity = 8,
                    Type = "Earrings",
                    MetalId = 1,
                    CategoryId = 4,
                    DiscountId = 2
                }
            );

            // Seed ProductGemstones
            modelBuilder.Entity<ProductGemstone>().HasData(
                new ProductGemstone { Id = 1, ProductId = 1, GemstoneId = 1, Carats = 1.0m },
                new ProductGemstone { Id = 2, ProductId = 2, GemstoneId = 4, Carats = 0.5m },
                new ProductGemstone { Id = 3, ProductId = 3, GemstoneId = 2, Carats = 0.75m },
                new ProductGemstone { Id = 4, ProductId = 4, GemstoneId = 1, Carats = 0.25m }
            );

            // Seed Customers
            modelBuilder.Entity<Customer>().HasData(
                new Customer
                {
                    Id = 1,
                    FirstName = "John",
                    LastName = "Doe",
                    Email = "john.doe@example.com",
                    Phone = "123-456-7890",
                    Address = "123 Main St"
                },
                new Customer
                {
                    Id = 2,
                    FirstName = "Jane",
                    LastName = "Smith",
                    Email = "jane.smith@example.com",
                    Phone = "987-654-3210",
                    Address = "456 Elm St"
                }
            );

            // Seed Reviews
            modelBuilder.Entity<Review>().HasData(
                new Review
                {
                    Id = 1,
                    Rating = 5,
                    Comment = "Beautiful ring, exactly as described!",
                    ReviewDate = new DateTime(2025, 4, 15),
                    ProductId = 1,
                    CustomerId = 1
                },
                new Review
                {
                    Id = 2,
                    Rating = 4,
                    Comment = "Nice necklace, but the clasp is a bit difficult to use.",
                    ReviewDate = new DateTime(2025, 4, 20),
                    ProductId = 2,
                    CustomerId = 2
                }
            );
        }
    }
}
```

## Creating and Applying Migrations

Now that we've updated our database schema, let's create and apply a new migration:

```bash
dotnet ef migrations add EnhancedSchema
dotnet ef database update
```

## Implementing Data Validation

Let's add data validation to our models to ensure data integrity. We'll use data annotations to define validation rules.

Update the `Product` class:

```csharp
using System.ComponentModel.DataAnnotations;

namespace JewelryJunction.Models
{
    public class Product
    {
        public int Id { get; set; }

        [Required]
        [StringLength(100)]
        public string Name { get; set; } = string.Empty;

        [StringLength(500)]
        public string Description { get; set; } = string.Empty;

        [Required]
        [Range(0.01, double.MaxValue, ErrorMessage = "Price must be greater than 0")]
        public decimal Price { get; set; }

        [Required]
        [Range(0, int.MaxValue, ErrorMessage = "Stock quantity cannot be negative")]
        public int StockQuantity { get; set; }

        [Required]
        [StringLength(50)]
        public string Type { get; set; } = string.Empty;

        // Relationships
        [Required]
        public int MetalId { get; set; }
        public Metal? Metal { get; set; }
        public List<ProductGemstone> ProductGemstones { get; set; } = new List<ProductGemstone>();

        [Required]
        public int CategoryId { get; set; }
        public Category? Category { get; set; }

        public int? DiscountId { get; set; }
        public Discount? Discount { get; set; }

        public List<Review> Reviews { get; set; } = new List<Review>();
    }
}
```

Apply similar validation to the other model classes.

## Querying Related Data

Now, let's update our endpoints to include related data in the responses. Update the `Program.cs` file:

```csharp
// Get all products with related data
app.MapGet("/products", async (JewelryContext db) =>
    await db.Products
        .Include(p => p.Metal)
        .Include(p => p.Category)
        .Include(p => p.Discount)
        .ToListAsync())
    .WithName("GetAllProducts")
    .WithOpenApi();

// Get product by ID with related data
app.MapGet("/products/{id}", async (int id, JewelryContext db) =>
{
    var product = await db.Products
        .Include(p => p.Metal)
        .Include(p => p.Category)
        .Include(p => p.Discount)
        .Include(p => p.ProductGemstones)
            .ThenInclude(pg => pg.Gemstone)
        .Include(p => p.Reviews)
            .ThenInclude(r => r.Customer)
        .FirstOrDefaultAsync(p => p.Id == id);

    return product != null ? Results.Ok(product) : Results.NotFound();
})
.WithName("GetProductById")
.WithOpenApi();

// Get products by category
app.MapGet("/categories/{id}/products", async (int id, JewelryContext db) =>
{
    var category = await db.Categories
        .Include(c => c.Products)
            .ThenInclude(p => p.Metal)
        .FirstOrDefaultAsync(c => c.Id == id);

    return category != null ? Results.Ok(category.Products) : Results.NotFound();
})
.WithName("GetProductsByCategory")
.WithOpenApi();

// Get products with discount
app.MapGet("/products/discounted", async (JewelryContext db) =>
    await db.Products
        .Include(p => p.Metal)
        .Include(p => p.Category)
        .Include(p => p.Discount)
        .Where(p => p.DiscountId != null && p.Discount.IsActive)
        .ToListAsync())
    .WithName("GetDiscountedProducts")
    .WithOpenApi();

// Get product reviews
app.MapGet("/products/{id}/reviews", async (int id, JewelryContext db) =>
{
    var product = await db.Products
        .Include(p => p.Reviews)
            .ThenInclude(r => r.Customer)
        .FirstOrDefaultAsync(p => p.Id == id);

    return product != null ? Results.Ok(product.Reviews) : Results.NotFound();
})
.WithName("GetProductReviews")
.WithOpenApi();
```

## Implementing Computed Properties

Let's add some computed properties to our models to provide additional information. Update the `Product` class:

```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace JewelryJunction.Models
{
    public class Product
    {
        // Existing properties...

        [NotMapped]
        public decimal DiscountedPrice
        {
            get
            {
                if (Discount != null && Discount.IsActive)
                {
                    return Price * (1 - Discount.DiscountPercent / 100);
                }
                return Price;
            }
        }

        [NotMapped]
        public decimal AverageRating
        {
            get
            {
                if (Reviews.Count == 0)
                {
                    return 0;
                }
                return (decimal)Reviews.Average(r => r.Rating);
            }
        }
    }
}
```

## Implementing Database Constraints

Let's add some database constraints to ensure data integrity at the database level. Update the `OnModelCreating` method in the `JewelryContext` class:

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Existing configuration...

    // Add constraints
    modelBuilder.Entity<Product>()
        .Property(p => p.Price)
        .HasColumnType("decimal(18,2)")
        .IsRequired();

    modelBuilder.Entity<Product>()
        .Property(p => p.Name)
        .IsRequired()
        .HasMaxLength(100);

    modelBuilder.Entity<Review>()
        .Property(r => r.Rating)
        .IsRequired()
        .HasAnnotation("Range", new[] { 1, 5 });

    modelBuilder.Entity<Discount>()
        .Property(d => d.DiscountPercent)
        .HasColumnType("decimal(5,2)")
        .IsRequired();

    // Seed data...
}
```

## Implementing Transactions

When performing multiple database operations that need to be atomic (all succeed or all fail), you should use transactions. Let's add an endpoint for creating an order that uses a transaction:

```csharp
// Create an order
app.MapPost("/orders", async (Order order, JewelryContext db) =>
{
    using var transaction = await db.Database.BeginTransactionAsync();

    try
    {
        // Add the order
        db.Orders.Add(order);
        await db.SaveChangesAsync();

        // Update product stock quantities
        foreach (var item in order.OrderItems)
        {
            var product = await db.Products.FindAsync(item.ProductId);
            if (product == null)
            {
                throw new Exception($"Product with ID {item.ProductId} not found");
            }

            if (product.StockQuantity < item.Quantity)
            {
                throw new Exception($"Not enough stock for product {product.Name}");
            }

            product.StockQuantity -= item.Quantity;
        }

        await db.SaveChangesAsync();

        // Commit the transaction
        await transaction.CommitAsync();

        return Results.Created($"/orders/{order.Id}", order);
    }
    catch (Exception ex)
    {
        // Rollback the transaction
        await transaction.RollbackAsync();
        return Results.Problem($"An error occurred: {ex.Message}");
    }
})
.WithName("CreateOrder")
.WithOpenApi();
```

## Conclusion

In this chapter, you've learned how to set up a more complex database schema for the Jewelry Junction API. You've implemented relationships between tables, added data validation and constraints, and used Entity Framework Core to query related data. You've also learned how to use transactions to ensure data integrity when performing multiple database operations.

In the next chapter, we'll implement endpoints for retrieving and managing orders, including creating new orders and updating order status.

## Practice Exercise

Enhance your Jewelry Junction database by:
1. Adding a `Supplier` entity with properties for `Id`, `Name`, `ContactName`, `Email`, and `Phone`
2. Creating a relationship between `Product` and `Supplier` (a product is supplied by one supplier)
3. Adding seed data for suppliers
4. Implementing validation for the `Supplier` entity
5. Creating endpoints for managing suppliers and retrieving products by supplier
6. Adding a computed property to the `Supplier` entity that returns the total value of products supplied by that supplier