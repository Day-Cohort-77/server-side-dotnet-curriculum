# Project Setup

In this chapter, we'll set up the Jewelry Junction project, a minimal API for managing a jewelry store's inventory and orders. This project will build on the concepts you've learned in the previous tracks, applying them to a new domain.

## Learning Objectives

By the end of this chapter, you should be able to:
- Set up a new minimal API project for Jewelry Junction
- Understand the project requirements and domain
- Define the initial data models
- Create a basic project structure
- Implement a simple endpoint to verify the setup

## Project Overview

Jewelry Junction is an online jewelry store that needs an API to manage its inventory and customer orders. The API will allow the store to:

- Manage products (rings, necklaces, bracelets, etc.)
- Track inventory
- Process customer orders
- Manage metals (gold, silver, platinum) and gemstones (diamonds, rubies, emeralds, etc.)
- Generate reports on sales and inventory

Throughout this track, you'll build this API step by step, learning how to implement various features and best practices.

## Setting Up the Project

Let's start by creating a new minimal API project for Jewelry Junction:

1. Open a terminal or command prompt
2. Navigate to the directory where you want to create your project
3. Run the following command:

```bash
dotnet new webapi -minimal -o JewelryJunction
```

This command creates a new minimal API project with the following parameters:
- `webapi`: The project template to use
- `-minimal`: A flag that indicates we want to create a minimal API
- `-o JewelryJunction`: The output directory for the project

4. Navigate to the project directory:

```bash
cd JewelryJunction
```

5. Open the project in Visual Studio Code:

```bash
code .
```

## Understanding the Domain

Before we start coding, let's understand the domain of our application. Jewelry Junction deals with:

- **Products**: The jewelry items for sale (rings, necklaces, bracelets, etc.)
- **Metals**: The materials used to make the jewelry (gold, silver, platinum, etc.)
- **Gemstones**: The precious stones used in the jewelry (diamonds, rubies, emeralds, etc.)
- **Orders**: Customer purchases of products
- **Customers**: People who place orders

Let's define the relationships between these entities:
- A Product is made of one Metal
- A Product can have zero or more Gemstones
- An Order contains one or more Products
- A Customer can place one or more Orders

## Defining the Data Models

Now, let's define our data models based on the domain. Create a new folder called `Models` and add the following files:

1. `Models/Product.cs`:

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
    }
}
```

2. `Models/Metal.cs`:

```csharp
namespace JewelryJunction.Models
{
    public class Metal
    {
        public int Id { get; set; }
        public string Name { get; set; } = string.Empty;
        public decimal PricePerGram { get; set; }

        // Relationships
        public List<Product> Products { get; set; } = new List<Product>();
    }
}
```

3. `Models/Gemstone.cs`:

```csharp
namespace JewelryJunction.Models
{
    public class Gemstone
    {
        public int Id { get; set; }
        public string Name { get; set; } = string.Empty;
        public decimal PricePerCarat { get; set; }

        // Relationships
        public List<ProductGemstone> ProductGemstones { get; set; } = new List<ProductGemstone>();
    }
}
```

4. `Models/ProductGemstone.cs` (a join entity for the many-to-many relationship between Products and Gemstones):

```csharp
namespace JewelryJunction.Models
{
    public class ProductGemstone
    {
        public int Id { get; set; }
        public decimal Carats { get; set; }

        // Relationships
        public int ProductId { get; set; }
        public Product? Product { get; set; }
        public int GemstoneId { get; set; }
        public Gemstone? Gemstone { get; set; }
    }
}
```

5. `Models/Order.cs`:

```csharp
namespace JewelryJunction.Models
{
    public class Order
    {
        public int Id { get; set; }
        public DateTime OrderDate { get; set; }
        public decimal TotalAmount { get; set; }
        public string Status { get; set; } = string.Empty; // Pending, Shipped, Delivered, Cancelled

        // Relationships
        public int CustomerId { get; set; }
        public Customer? Customer { get; set; }
        public List<OrderItem> OrderItems { get; set; } = new List<OrderItem>();
    }
}
```

6. `Models/OrderItem.cs`:

```csharp
namespace JewelryJunction.Models
{
    public class OrderItem
    {
        public int Id { get; set; }
        public int Quantity { get; set; }
        public decimal UnitPrice { get; set; }

        // Relationships
        public int OrderId { get; set; }
        public Order? Order { get; set; }
        public int ProductId { get; set; }
        public Product? Product { get; set; }
    }
}
```

7. `Models/Customer.cs`:

```csharp
namespace JewelryJunction.Models
{
    public class Customer
    {
        public int Id { get; set; }
        public string FirstName { get; set; } = string.Empty;
        public string LastName { get; set; } = string.Empty;
        public string Email { get; set; } = string.Empty;
        public string Phone { get; set; } = string.Empty;
        public string Address { get; set; } = string.Empty;

        // Relationships
        public List<Order> Orders { get; set; } = new List<Order>();
    }
}
```

## Setting Up the Database Context

Now, let's create a database context for our application. Create a new folder called `Data` and add the following file:

1. `Data/JewelryContext.cs`:

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
        }
    }
}
```

## Configuring the Database Connection

Now, let's configure the database connection in our `appsettings.json` file:

1. Update the `appsettings.json` file to include the connection string:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Host=localhost;Database=jewelryjunction;Username=postgres;Password=your_password"
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

## Updating the Program.cs File

Now, let's update the `Program.cs` file to configure the database context and define some basic endpoints:

```csharp
using Microsoft.EntityFrameworkCore;
using JewelryJunction.Data;
using JewelryJunction.Models;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
// Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Add database context
builder.Services.AddDbContext<JewelryContext>(options =>
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
app.MapGet("/", () => "Welcome to Jewelry Junction API!");

// Get all products
app.MapGet("/products", async (JewelryContext db) =>
    await db.Products.ToListAsync())
    .WithName("GetAllProducts")
    .WithOpenApi();

// Get product by ID
app.MapGet("/products/{id}", async (int id, JewelryContext db) =>
{
    var product = await db.Products
        .Include(p => p.Metal)
        .Include(p => p.ProductGemstones)
            .ThenInclude(pg => pg.Gemstone)
        .FirstOrDefaultAsync(p => p.Id == id);

    return product != null ? Results.Ok(product) : Results.NotFound();
})
.WithName("GetProductById")
.WithOpenApi();

app.Run();
```

## Creating Migrations

Now, let's create and apply migrations to set up the database:

1. Install the EF Core CLI tools (if you haven't already):

```bash
dotnet tool install --global dotnet-ef
```

2. Add the required packages:

```bash
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
```

3. Create the initial migration:

```bash
dotnet ef migrations add InitialCreate
```

4. Apply the migration to create the database:

```bash
dotnet ef database update
```

## Seeding the Database

Let's add some seed data to our database. Update the `OnModelCreating` method in the `JewelryContext` class:

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Configure relationships (existing code)

    // Seed data
    modelBuilder.Entity<Metal>().HasData(
        new Metal { Id = 1, Name = "Gold", PricePerGram = 60.00m },
        new Metal { Id = 2, Name = "Silver", PricePerGram = 0.85m },
        new Metal { Id = 3, Name = "Platinum", PricePerGram = 30.00m }
    );

    modelBuilder.Entity<Gemstone>().HasData(
        new Gemstone { Id = 1, Name = "Diamond", PricePerCarat = 1500.00m },
        new Gemstone { Id = 2, Name = "Ruby", PricePerCarat = 1000.00m },
        new Gemstone { Id = 3, Name = "Emerald", PricePerCarat = 800.00m },
        new Gemstone { Id = 4, Name = "Sapphire", PricePerCarat = 1200.00m }
    );

    modelBuilder.Entity<Product>().HasData(
        new Product { Id = 1, Name = "Diamond Ring", Description = "A beautiful diamond ring", Price = 1000.00m, StockQuantity = 10, Type = "Ring", MetalId = 1 },
        new Product { Id = 2, Name = "Silver Necklace", Description = "An elegant silver necklace", Price = 150.00m, StockQuantity = 15, Type = "Necklace", MetalId = 2 },
        new Product { Id = 3, Name = "Platinum Bracelet", Description = "A luxurious platinum bracelet", Price = 800.00m, StockQuantity = 5, Type = "Bracelet", MetalId = 3 }
    );

    modelBuilder.Entity<ProductGemstone>().HasData(
        new ProductGemstone { Id = 1, ProductId = 1, GemstoneId = 1, Carats = 1.0m }
    );

    modelBuilder.Entity<Customer>().HasData(
        new Customer { Id = 1, FirstName = "John", LastName = "Doe", Email = "john.doe@example.com", Phone = "123-456-7890", Address = "123 Main St" },
        new Customer { Id = 2, FirstName = "Jane", LastName = "Smith", Email = "jane.smith@example.com", Phone = "987-654-3210", Address = "456 Elm St" }
    );
}
```

After updating the `OnModelCreating` method, create a new migration and apply it:

```bash
dotnet ef migrations add SeedData
dotnet ef database update
```

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

## Testing the API

Let's test the API using Swagger:

1. Click on the `GET /products` endpoint
2. Click the "Try it out" button
3. Click the "Execute" button
4. You should see a response with the list of products

You can also test the `GET /products/{id}` endpoint by providing a product ID (e.g., 1).

## Conclusion

In this chapter, you've set up the Jewelry Junction project, defined the data models, created a database context, and implemented basic endpoints. You've also seeded the database with initial data and tested the API.

In the next chapter, we'll expand the API by implementing endpoints for managing the jewelry database, including creating, updating, and deleting products.

## Practice Exercise

Enhance your Jewelry Junction API by:
1. Adding endpoints for managing metals (GET, POST, PUT, DELETE)
2. Adding endpoints for managing gemstones (GET, POST, PUT, DELETE)
3. Implementing a search endpoint that allows filtering products by type, metal, or price range
4. Adding validation to ensure that product prices are positive and stock quantities are non-negative
5. Creating a simple endpoint that returns statistics about the inventory (total number of products, average price, etc.)