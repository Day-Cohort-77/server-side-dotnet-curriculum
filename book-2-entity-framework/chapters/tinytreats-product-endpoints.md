# Product Management Endpoints

In this chapter, we'll implement product management endpoints for our TinyTreats application. These endpoints will allow administrators to create, read, update, and delete bakery products, while customers can browse the available products.

## Understanding Product Management

Product management is a core feature of our bakery management system. It involves:

1. **Creating products**: Adding new bakery items to the catalog
2. **Reading products**: Browsing and searching for products
3. **Updating products**: Modifying product details and availability
4. **Deleting products**: Removing products from the catalog

We'll implement these operations as RESTful endpoints, following our organized approach of placing related endpoints in separate files.

## Creating the Product Endpoints

Let's create a new file called `Endpoints/ProductEndpoints.cs` to contain all our product-related endpoints:

```csharp
// Endpoints/ProductEndpoints.cs
using Microsoft.EntityFrameworkCore;
using TinyTreats.Data;
using TinyTreats.DTO;
using TinyTreats.Models;

namespace TinyTreats.Endpoints;

public static class ProductEndpoints
{
    public static void MapProductEndpoints(this WebApplication app)
    {
        // Get all products - Public
        app.MapGet("/products", async (TinyTreatsDbContext dbContext) =>
        {
            var products = await dbContext.Products
                .Where(p => p.IsAvailable)
                .Select(p => new ProductDto
                {
                    Id = p.Id,
                    Name = p.Name,
                    Description = p.Description,
                    Price = p.Price,
                    IsAvailable = p.IsAvailable,
                    ImageUrl = p.ImageUrl
                })
                .ToListAsync();

            return Results.Ok(products);
        });

        // Get product by ID - Public
        app.MapGet("/products/{id}", async (int id, TinyTreatsDbContext dbContext) =>
        {
            var product = await dbContext.Products.FindAsync(id);

            if (product == null)
            {
                return Results.NotFound();
            }

            // Only return available products to the public
            if (!product.IsAvailable)
            {
                return Results.NotFound();
            }

            return Results.Ok(new ProductDto
            {
                Id = product.Id,
                Name = product.Name,
                Description = product.Description,
                Price = product.Price,
                IsAvailable = product.IsAvailable,
                ImageUrl = product.ImageUrl
            });
        });

        // Get all products (including unavailable) - Admin only
        app.MapGet("/admin/products", async (TinyTreatsDbContext dbContext) =>
        {
            var products = await dbContext.Products
                .Select(p => new ProductDto
                {
                    Id = p.Id,
                    Name = p.Name,
                    Description = p.Description,
                    Price = p.Price,
                    IsAvailable = p.IsAvailable,
                    ImageUrl = p.ImageUrl
                })
                .ToListAsync();

            return Results.Ok(products);
        }).RequireAuthorization(policy => policy.RequireRole("Admin"));

        // Create a new product - Admin only
        app.MapPost("/products", async (
            ProductCreateDto productDto,
            TinyTreatsDbContext dbContext) =>
        {
            var product = new Product
            {
                Name = productDto.Name,
                Description = productDto.Description,
                Price = productDto.Price,
                IsAvailable = productDto.IsAvailable,
                ImageUrl = productDto.ImageUrl
            };

            dbContext.Products.Add(product);
            await dbContext.SaveChangesAsync();

            return Results.Created($"/products/{product.Id}", new ProductDto
            {
                Id = product.Id,
                Name = product.Name,
                Description = product.Description,
                Price = product.Price,
                IsAvailable = product.IsAvailable,
                ImageUrl = product.ImageUrl
            });
        }).RequireAuthorization(policy => policy.RequireRole("Admin"));

        // Update a product - Admin only
        app.MapPut("/products/{id}", async (
            int id,
            ProductUpdateDto productDto,
            TinyTreatsDbContext dbContext) =>
        {
            var product = await dbContext.Products.FindAsync(id);

            if (product == null)
            {
                return Results.NotFound();
            }

            // Update only the properties that are provided
            if (productDto.Name != null)
            {
                product.Name = productDto.Name;
            }

            if (productDto.Description != null)
            {
                product.Description = productDto.Description;
            }

            if (productDto.Price.HasValue)
            {
                product.Price = productDto.Price.Value;
            }

            if (productDto.IsAvailable.HasValue)
            {
                product.IsAvailable = productDto.IsAvailable.Value;
            }

            if (productDto.ImageUrl != null)
            {
                product.ImageUrl = productDto.ImageUrl;
            }

            await dbContext.SaveChangesAsync();

            return Results.Ok(new ProductDto
            {
                Id = product.Id,
                Name = product.Name,
                Description = product.Description,
                Price = product.Price,
                IsAvailable = product.IsAvailable,
                ImageUrl = product.ImageUrl
            });
        }).RequireAuthorization(policy => policy.RequireRole("Admin"));

        // Delete a product - Admin only
        app.MapDelete("/products/{id}", async (
            int id,
            TinyTreatsDbContext dbContext) =>
        {
            var product = await dbContext.Products.FindAsync(id);

            if (product == null)
            {
                return Results.NotFound();
            }

            // Check if the product is referenced by any order items
            var hasOrderItems = await dbContext.OrderItems
                .AnyAsync(oi => oi.ProductId == id);

            if (hasOrderItems)
            {
                // Instead of deleting, mark as unavailable
                product.IsAvailable = false;
                await dbContext.SaveChangesAsync();
                return Results.Ok(new { message = "Product marked as unavailable because it has order references." });
            }

            // If no order items reference this product, we can safely delete it
            dbContext.Products.Remove(product);
            await dbContext.SaveChangesAsync();

            return Results.NoContent();
        }).RequireAuthorization(policy => policy.RequireRole("Admin"));
    }
}
```

## Understanding the Product Endpoints

Let's break down each endpoint:

### Get All Products (Public)

The get all products endpoint (`/products`) returns all available products:

```csharp
app.MapGet("/products", async (TinyTreatsDbContext dbContext) =>
{
    var products = await dbContext.Products
        .Where(p => p.IsAvailable)
        .Select(p => new ProductDto
        {
            Id = p.Id,
            Name = p.Name,
            Description = p.Description,
            Price = p.Price,
            IsAvailable = p.IsAvailable,
            ImageUrl = p.ImageUrl
        })
        .ToListAsync();

    return Results.Ok(products);
});
```

This endpoint:
1. Retrieves all available products from the database
2. Maps them to `ProductDto` objects
3. Returns a 200 OK response with the products
4. Is publicly accessible (no authorization required)

### Get Product by ID (Public)

The get product by ID endpoint (`/products/{id}`) returns a specific product:

```csharp
app.MapGet("/products/{id}", async (int id, TinyTreatsDbContext dbContext) =>
{
    var product = await dbContext.Products.FindAsync(id);

    if (product == null)
    {
        return Results.NotFound();
    }

    // Only return available products to the public
    if (!product.IsAvailable)
    {
        return Results.NotFound();
    }

    return Results.Ok(new ProductDto
    {
        Id = product.Id,
        Name = product.Name,
        Description = product.Description,
        Price = product.Price,
        IsAvailable = product.IsAvailable,
        ImageUrl = product.ImageUrl
    });
});
```

This endpoint:
1. Retrieves a specific product by ID
2. Returns a 404 Not Found response if the product doesn't exist or isn't available
3. Maps the product to a `ProductDto` object
4. Returns a 200 OK response with the product
5. Is publicly accessible

### Get All Products (Admin)

The get all products (admin) endpoint (`/admin/products`) returns all products, including unavailable ones:

```csharp
app.MapGet("/admin/products", async (TinyTreatsDbContext dbContext) =>
{
    var products = await dbContext.Products
        .Select(p => new ProductDto
        {
            Id = p.Id,
            Name = p.Name,
            Description = p.Description,
            Price = p.Price,
            IsAvailable = p.IsAvailable,
            ImageUrl = p.ImageUrl
        })
        .ToListAsync();

    return Results.Ok(products);
}).RequireAuthorization(policy => policy.RequireRole("Admin"));
```

This endpoint:
1. Retrieves all products from the database, including unavailable ones
2. Maps them to `ProductDto` objects
3. Returns a 200 OK response with the products
4. Requires the "Admin" role

### Create Product

The create product endpoint (`/products`) creates a new product:

```csharp
app.MapPost("/products", async (
    ProductCreateDto productDto,
    TinyTreatsDbContext dbContext) =>
{
    var product = new Product
    {
        Name = productDto.Name,
        Description = productDto.Description,
        Price = productDto.Price,
        IsAvailable = productDto.IsAvailable,
        ImageUrl = productDto.ImageUrl
    };

    dbContext.Products.Add(product);
    await dbContext.SaveChangesAsync();

    return Results.Created($"/products/{product.Id}", new ProductDto
    {
        Id = product.Id,
        Name = product.Name,
        Description = product.Description,
        Price = product.Price,
        IsAvailable = product.IsAvailable,
        ImageUrl = product.ImageUrl
    });
}).RequireAuthorization(policy => policy.RequireRole("Admin"));
```

This endpoint:
1. Creates a new product from the provided data
2. Adds it to the database
3. Returns a 201 Created response with the created product
4. Requires the "Admin" role

### Update Product

The update product endpoint (`/products/{id}`) updates an existing product:

```csharp
app.MapPut("/products/{id}", async (
    int id,
    ProductUpdateDto productDto,
    TinyTreatsDbContext dbContext) =>
{
    var product = await dbContext.Products.FindAsync(id);

    if (product == null)
    {
        return Results.NotFound();
    }

    // Update only the properties that are provided
    if (productDto.Name != null)
    {
        product.Name = productDto.Name;
    }

    if (productDto.Description != null)
    {
        product.Description = productDto.Description;
    }

    if (productDto.Price.HasValue)
    {
        product.Price = productDto.Price.Value;
    }

    if (productDto.IsAvailable.HasValue)
    {
        product.IsAvailable = productDto.IsAvailable.Value;
    }

    if (productDto.ImageUrl != null)
    {
        product.ImageUrl = productDto.ImageUrl;
    }

    await dbContext.SaveChangesAsync();

    return Results.Ok(new ProductDto
    {
        Id = product.Id,
        Name = product.Name,
        Description = product.Description,
        Price = product.Price,
        IsAvailable = product.IsAvailable,
        ImageUrl = product.ImageUrl
    });
}).RequireAuthorization(policy => policy.RequireRole("Admin"));
```

This endpoint:
1. Retrieves the product by ID
2. Returns a 404 Not Found response if the product doesn't exist
3. Updates only the properties that are provided in the request
4. Saves the changes to the database
5. Returns a 200 OK response with the updated product
6. Requires the "Admin" role

### Delete Product

The delete product endpoint (`/products/{id}`) deletes a product or marks it as unavailable:

```csharp
app.MapDelete("/products/{id}", async (
    int id,
    TinyTreatsDbContext dbContext) =>
{
    var product = await dbContext.Products.FindAsync(id);

    if (product == null)
    {
        return Results.NotFound();
    }

    // Check if the product is referenced by any order items
    var hasOrderItems = await dbContext.OrderItems
        .AnyAsync(oi => oi.ProductId == id);

    if (hasOrderItems)
    {
        // Instead of deleting, mark as unavailable
        product.IsAvailable = false;
        await dbContext.SaveChangesAsync();
        return Results.Ok(new { message = "Product marked as unavailable because it has order references." });
    }

    // If no order items reference this product, we can safely delete it
    dbContext.Products.Remove(product);
    await dbContext.SaveChangesAsync();

    return Results.NoContent();
}).RequireAuthorization(policy => policy.RequireRole("Admin"));
```

This endpoint:
1. Retrieves the product by ID
2. Returns a 404 Not Found response if the product doesn't exist
3. Checks if the product is referenced by any order items
4. If it is, marks the product as unavailable instead of deleting it
5. If it's not, deletes the product from the database
6. Returns a 204 No Content response if the product was deleted, or a 200 OK response with a message if it was marked as unavailable
7. Requires the "Admin" role

## Authorization Patterns

Notice the different authorization patterns we're using:

1. **Public endpoints**: No authorization required
   ```csharp
   app.MapGet("/products", ...);
   ```

2. **Admin-only endpoints**: Require the "Admin" role
   ```csharp
   app.MapPost("/products", ...).RequireAuthorization(policy => policy.RequireRole("Admin"));
   ```

This allows us to control access to our endpoints based on the user's role.

## Testing Product Management

You can test these endpoints using a tool like Yaak:

1. Get all products (public):
   ```http
   GET /products
   ```

2. Get a specific product (public):
   ```http
   GET /products/1
   ```

3. Login as an admin:
   ```http
   POST /auth/login
   Content-Type: application/json

   {
     "email": "admin@tinytreats.com",
     "password": "Admin123!"
   }
   ```

4. Get all products (admin):
   ```http
   GET /admin/products
   ```

5. Create a new product:
   ```http
   POST /products
   Content-Type: application/json

   {
     "name": "Red Velvet Cupcake",
     "description": "Delicious red velvet cupcake with cream cheese frosting",
     "price": 3.99,
     "isAvailable": true,
     "imageUrl": "/images/red-velvet-cupcake.jpg"
   }
   ```

6. Update a product:
   ```http
   PUT /products/1
   Content-Type: application/json

   {
     "price": 2.75,
     "isAvailable": false
   }
   ```

7. Delete a product:
   ```http
   DELETE /products/1
   ```

## Summary

In this chapter, we've implemented product management endpoints for our TinyTreats application:

- Get all products endpoint (public) for browsing available products
- Get product by ID endpoint (public) for viewing a specific product
- Get all products endpoint (admin) for viewing all products, including unavailable ones
- Create product endpoint for adding new products
- Update product endpoint for modifying existing products
- Delete product endpoint for removing products or marking them as unavailable

These endpoints provide a complete product management system for our bakery application. In the next chapter, we'll implement order management endpoints to allow customers to place orders and staff to manage them.

[Next: Order Management Endpoints](./tinytreats-order-endpoints.md)