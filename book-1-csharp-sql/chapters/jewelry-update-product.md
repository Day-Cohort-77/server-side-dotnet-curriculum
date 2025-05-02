# Update a Product

In this chapter, we'll implement the endpoint for updating a product in our Jewelry Junction API. This will involve validation, handling of related data, and ensuring data consistency.

## Learning Objectives

By the end of this chapter, you should be able to:
- Implement a PUT endpoint to update a product
- Validate the product data before updating
- Handle updates to related data
- Implement proper error handling
- Return appropriate responses

## Understanding the Product Update Process

Updating a product involves several considerations:
1. Validating that the product exists
2. Validating the updated data
3. Handling updates to related data (e.g., gemstones)
4. Ensuring data consistency
5. Returning the updated product

Let's implement an endpoint that addresses these considerations.

## Implementing the Basic Endpoint

Let's start by implementing a basic endpoint for updating a product:

```csharp
// Update a product
app.MapPut("/products/{id}", async (int id, Product updatedProduct, JewelryContext db) =>
{
    try
    {
        // Find the product
        var product = await db.Products.FindAsync(id);

        if (product == null)
        {
            return Results.NotFound(new { Message = $"Product with ID {id} not found" });
        }

        // Update product properties
        product.Name = updatedProduct.Name;
        product.Description = updatedProduct.Description;
        product.Price = updatedProduct.Price;
        product.StockQuantity = updatedProduct.StockQuantity;
        product.Type = updatedProduct.Type;
        product.MetalId = updatedProduct.MetalId;
        product.CategoryId = updatedProduct.CategoryId;
        product.DiscountId = updatedProduct.DiscountId;

        // Save changes
        await db.SaveChangesAsync();

        return Results.Ok(product);
    }
    catch (Exception ex)
    {
        // Log the exception
        Console.WriteLine($"Error updating product: {ex.Message}");

        // Return a 500 Internal Server Error response
        return Results.Problem(
            title: "An error occurred while updating the product",
            detail: ex.Message,
            statusCode: 500
        );
    }
})
.WithName("UpdateProduct")
.WithOpenApi();
```

This basic endpoint:
1. Finds the product by ID
2. Updates the product properties with the values from the request
3. Saves the changes to the database
4. Returns the updated product

However, this implementation has several limitations:
- It doesn't validate the updated data
- It doesn't handle updates to related data (e.g., gemstones)
- It doesn't check if the related entities (metal, category, discount) exist

Let's enhance the endpoint to address these limitations.

## Implementing Data Validation

First, let's add validation for the updated data:

```csharp
// Update a product with validation
app.MapPut("/products/{id}", async (int id, Product updatedProduct, JewelryContext db) =>
{
    try
    {
        // Find the product
        var product = await db.Products.FindAsync(id);

        if (product == null)
        {
            return Results.NotFound(new { Message = $"Product with ID {id} not found" });
        }

        // Validate the updated data
        if (string.IsNullOrWhiteSpace(updatedProduct.Name))
        {
            return Results.BadRequest(new { Message = "Product name is required" });
        }

        if (updatedProduct.Price <= 0)
        {
            return Results.BadRequest(new { Message = "Product price must be greater than 0" });
        }

        if (updatedProduct.StockQuantity < 0)
        {
            return Results.BadRequest(new { Message = "Product stock quantity cannot be negative" });
        }

        // Validate that the related entities exist
        var metalExists = await db.Metals.AnyAsync(m => m.Id == updatedProduct.MetalId);
        if (!metalExists)
        {
            return Results.BadRequest(new { Message = $"Metal with ID {updatedProduct.MetalId} not found" });
        }

        var categoryExists = await db.Categories.AnyAsync(c => c.Id == updatedProduct.CategoryId);
        if (!categoryExists)
        {
            return Results.BadRequest(new { Message = $"Category with ID {updatedProduct.CategoryId} not found" });
        }

        if (updatedProduct.DiscountId.HasValue)
        {
            var discountExists = await db.Discounts.AnyAsync(d => d.Id == updatedProduct.DiscountId.Value);
            if (!discountExists)
            {
                return Results.BadRequest(new { Message = $"Discount with ID {updatedProduct.DiscountId.Value} not found" });
            }
        }

        // Update product properties
        product.Name = updatedProduct.Name;
        product.Description = updatedProduct.Description;
        product.Price = updatedProduct.Price;
        product.StockQuantity = updatedProduct.StockQuantity;
        product.Type = updatedProduct.Type;
        product.MetalId = updatedProduct.MetalId;
        product.CategoryId = updatedProduct.CategoryId;
        product.DiscountId = updatedProduct.DiscountId;

        // Save changes
        await db.SaveChangesAsync();

        return Results.Ok(product);
    }
    catch (Exception ex)
    {
        // Log the exception
        Console.WriteLine($"Error updating product: {ex.Message}");

        // Return a 500 Internal Server Error response
        return Results.Problem(
            title: "An error occurred while updating the product",
            detail: ex.Message,
            statusCode: 500
        );
    }
})
.WithName("UpdateProduct")
.WithOpenApi();
```

Now, the endpoint validates:
- That the product name is not empty
- That the product price is greater than 0
- That the product stock quantity is not negative
- That the related entities (metal, category, discount) exist

## Handling Related Data

Next, let's enhance the endpoint to handle updates to related data, specifically the gemstones associated with the product:

```csharp
// Update a product with related data
app.MapPut("/products/{id}", async (int id, ProductUpdateRequest request, JewelryContext db) =>
{
    using var transaction = await db.Database.BeginTransactionAsync();

    try
    {
        // Find the product with its gemstones
        var product = await db.Products
            .Include(p => p.ProductGemstones)
            .FirstOrDefaultAsync(p => p.Id == id);

        if (product == null)
        {
            return Results.NotFound(new { Message = $"Product with ID {id} not found" });
        }

        // Validate the updated data
        if (string.IsNullOrWhiteSpace(request.Name))
        {
            return Results.BadRequest(new { Message = "Product name is required" });
        }

        if (request.Price <= 0)
        {
            return Results.BadRequest(new { Message = "Product price must be greater than 0" });
        }

        if (request.StockQuantity < 0)
        {
            return Results.BadRequest(new { Message = "Product stock quantity cannot be negative" });
        }

        // Validate that the related entities exist
        var metalExists = await db.Metals.AnyAsync(m => m.Id == request.MetalId);
        if (!metalExists)
        {
            return Results.BadRequest(new { Message = $"Metal with ID {request.MetalId} not found" });
        }

        var categoryExists = await db.Categories.AnyAsync(c => c.Id == request.CategoryId);
        if (!categoryExists)
        {
            return Results.BadRequest(new { Message = $"Category with ID {request.CategoryId} not found" });
        }

        if (request.DiscountId.HasValue)
        {
            var discountExists = await db.Discounts.AnyAsync(d => d.Id == request.DiscountId.Value);
            if (!discountExists)
            {
                return Results.BadRequest(new { Message = $"Discount with ID {request.DiscountId.Value} not found" });
            }
        }

        // Update product properties
        product.Name = request.Name;
        product.Description = request.Description;
        product.Price = request.Price;
        product.StockQuantity = request.StockQuantity;
        product.Type = request.Type;
        product.MetalId = request.MetalId;
        product.CategoryId = request.CategoryId;
        product.DiscountId = request.DiscountId;

        // Handle gemstones
        if (request.Gemstones != null)
        {
            // Remove existing gemstones
            db.ProductGemstones.RemoveRange(product.ProductGemstones);

            // Add new gemstones
            foreach (var gemstoneRequest in request.Gemstones)
            {
                // Validate that the gemstone exists
                var gemstoneExists = await db.Gemstones.AnyAsync(g => g.Id == gemstoneRequest.GemstoneId);
                if (!gemstoneExists)
                {
                    return Results.BadRequest(new { Message = $"Gemstone with ID {gemstoneRequest.GemstoneId} not found" });
                }

                // Validate carats
                if (gemstoneRequest.Carats <= 0)
                {
                    return Results.BadRequest(new { Message = "Gemstone carats must be greater than 0" });
                }

                // Add the gemstone to the product
                var productGemstone = new ProductGemstone
                {
                    ProductId = product.Id,
                    GemstoneId = gemstoneRequest.GemstoneId,
                    Carats = gemstoneRequest.Carats
                };

                product.ProductGemstones.Add(productGemstone);
            }
        }

        // Save changes
        await db.SaveChangesAsync();

        // Commit the transaction
        await transaction.CommitAsync();

        // Return the updated product with its gemstones
        var updatedProduct = await db.Products
            .Include(p => p.Metal)
            .Include(p => p.Category)
            .Include(p => p.Discount)
            .Include(p => p.ProductGemstones)
                .ThenInclude(pg => pg.Gemstone)
            .FirstOrDefaultAsync(p => p.Id == id);

        return Results.Ok(updatedProduct);
    }
    catch (Exception ex)
    {
        // Rollback the transaction
        await transaction.RollbackAsync();

        // Log the exception
        Console.WriteLine($"Error updating product: {ex.Message}");

        // Return a 500 Internal Server Error response
        return Results.Problem(
            title: "An error occurred while updating the product",
            detail: ex.Message,
            statusCode: 500
        );
    }
})
.WithName("UpdateProduct")
.WithOpenApi();

// Product update request model
public class ProductUpdateRequest
{
    public string Name { get; set; } = string.Empty;
    public string Description { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public int StockQuantity { get; set; }
    public string Type { get; set; } = string.Empty;
    public int MetalId { get; set; }
    public int CategoryId { get; set; }
    public int? DiscountId { get; set; }
    public List<GemstoneRequest>? Gemstones { get; set; }
}

public class GemstoneRequest
{
    public int GemstoneId { get; set; }
    public decimal Carats { get; set; }
}
```

In this enhanced endpoint:
1. We use a transaction to ensure data consistency
2. We include the product's gemstones when retrieving it
3. We use a custom request model (`ProductUpdateRequest`) that includes a list of gemstones
4. We handle updates to gemstones by removing the existing ones and adding the new ones
5. We validate that the gemstones exist and that the carats are valid
6. We return the updated product with all its related data

## Formatting the Response

Let's enhance the endpoint further to format the response in a more user-friendly way:

```csharp
// Update a product with formatted response
app.MapPut("/products/{id}", async (int id, ProductUpdateRequest request, JewelryContext db) =>
{
    using var transaction = await db.Database.BeginTransactionAsync();

    try
    {
        // Find the product with its gemstones
        var product = await db.Products
            .Include(p => p.ProductGemstones)
            .FirstOrDefaultAsync(p => p.Id == id);

        if (product == null)
        {
            return Results.NotFound(new { Message = $"Product with ID {id} not found" });
        }

        // Validate the updated data
        if (string.IsNullOrWhiteSpace(request.Name))
        {
            return Results.BadRequest(new { Message = "Product name is required" });
        }

        if (request.Price <= 0)
        {
            return Results.BadRequest(new { Message = "Product price must be greater than 0" });
        }

        if (request.StockQuantity < 0)
        {
            return Results.BadRequest(new { Message = "Product stock quantity cannot be negative" });
        }

        // Validate that the related entities exist
        var metalExists = await db.Metals.AnyAsync(m => m.Id == request.MetalId);
        if (!metalExists)
        {
            return Results.BadRequest(new { Message = $"Metal with ID {request.MetalId} not found" });
        }

        var categoryExists = await db.Categories.AnyAsync(c => c.Id == request.CategoryId);
        if (!categoryExists)
        {
            return Results.BadRequest(new { Message = $"Category with ID {request.CategoryId} not found" });
        }

        if (request.DiscountId.HasValue)
        {
            var discountExists = await db.Discounts.AnyAsync(d => d.Id == request.DiscountId.Value);
            if (!discountExists)
            {
                return Results.BadRequest(new { Message = $"Discount with ID {request.DiscountId.Value} not found" });
            }
        }

        // Update product properties
        product.Name = request.Name;
        product.Description = request.Description;
        product.Price = request.Price;
        product.StockQuantity = request.StockQuantity;
        product.Type = request.Type;
        product.MetalId = request.MetalId;
        product.CategoryId = request.CategoryId;
        product.DiscountId = request.DiscountId;

        // Handle gemstones
        if (request.Gemstones != null)
        {
            // Remove existing gemstones
            db.ProductGemstones.RemoveRange(product.ProductGemstones);

            // Add new gemstones
            foreach (var gemstoneRequest in request.Gemstones)
            {
                // Validate that the gemstone exists
                var gemstoneExists = await db.Gemstones.AnyAsync(g => g.Id == gemstoneRequest.GemstoneId);
                if (!gemstoneExists)
                {
                    return Results.BadRequest(new { Message = $"Gemstone with ID {gemstoneRequest.GemstoneId} not found" });
                }

                // Validate carats
                if (gemstoneRequest.Carats <= 0)
                {
                    return Results.BadRequest(new { Message = "Gemstone carats must be greater than 0" });
                }

                // Add the gemstone to the product
                var productGemstone = new ProductGemstone
                {
                    ProductId = product.Id,
                    GemstoneId = gemstoneRequest.GemstoneId,
                    Carats = gemstoneRequest.Carats
                };

                product.ProductGemstones.Add(productGemstone);
            }
        }

        // Save changes
        await db.SaveChangesAsync();

        // Commit the transaction
        await transaction.CommitAsync();

        // Retrieve the updated product with all related data
        var updatedProduct = await db.Products
            .AsNoTracking()
            .Include(p => p.Metal)
            .Include(p => p.Category)
            .Include(p => p.Discount)
            .Include(p => p.ProductGemstones)
                .ThenInclude(pg => pg.Gemstone)
            .FirstOrDefaultAsync(p => p.Id == id);

        // Format the response
        var response = new
        {
            updatedProduct.Id,
            updatedProduct.Name,
            updatedProduct.Description,
            updatedProduct.Price,
            DiscountedPrice = updatedProduct.Discount != null && updatedProduct.Discount.IsActive
                ? updatedProduct.Price * (1 - updatedProduct.Discount.DiscountPercent / 100)
                : (decimal?)null,
            updatedProduct.StockQuantity,
            updatedProduct.Type,
            Metal = new
            {
                updatedProduct.Metal.Id,
                updatedProduct.Metal.Name,
                updatedProduct.Metal.PricePerGram
            },
            Category = new
            {
                updatedProduct.Category.Id,
                updatedProduct.Category.Name,
                updatedProduct.Category.Description
            },
            Discount = updatedProduct.Discount != null ? new
            {
                updatedProduct.Discount.Id,
                updatedProduct.Discount.Name,
                updatedProduct.Discount.Description,
                updatedProduct.Discount.DiscountPercent,
                updatedProduct.Discount.StartDate,
                updatedProduct.Discount.EndDate,
                updatedProduct.Discount.IsActive
            } : null,
            Gemstones = updatedProduct.ProductGemstones.Select(pg => new
            {
                Id = pg.Gemstone.Id,
                Name = pg.Gemstone.Name,
                PricePerCarat = pg.Gemstone.PricePerCarat,
                Carats = pg.Carats,
                TotalValue = pg.Carats * pg.Gemstone.PricePerCarat
            }).ToList(),
            UpdatedAt = DateTime.Now
        };

        return Results.Ok(response);
    }
    catch (Exception ex)
    {
        // Rollback the transaction
        await transaction.RollbackAsync();

        // Log the exception
        Console.WriteLine($"Error updating product: {ex.Message}");

        // Return a 500 Internal Server Error response
        return Results.Problem(
            title: "An error occurred while updating the product",
            detail: ex.Message,
            statusCode: 500
        );
    }
})
.WithName("UpdateProduct")
.WithOpenApi();
```

Now, the endpoint returns a well-formatted response that includes:
- Basic product information
- Calculated fields like discounted price
- Formatted information about related entities (metal, category, discount)
- Detailed information about gemstones, including the total value of each gemstone
- A timestamp indicating when the update occurred

## Implementing Partial Updates

In many cases, clients may want to update only specific fields of a product, rather than providing all fields. Let's implement a PATCH endpoint for partial updates:

```csharp
// Partially update a product
app.MapPatch("/products/{id}", async (int id, JsonPatchDocument<Product> patchDoc, JewelryContext db) =>
{
    try
    {
        // Find the product
        var product = await db.Products.FindAsync(id);

        if (product == null)
        {
            return Results.NotFound(new { Message = $"Product with ID {id} not found" });
        }

        // Apply the patch
        patchDoc.ApplyTo(product);

        // Validate the updated product
        if (string.IsNullOrWhiteSpace(product.Name))
        {
            return Results.BadRequest(new { Message = "Product name is required" });
        }

        if (product.Price <= 0)
        {
            return Results.BadRequest(new { Message = "Product price must be greater than 0" });
        }

        if (product.StockQuantity < 0)
        {
            return Results.BadRequest(new { Message = "Product stock quantity cannot be negative" });
        }

        // Validate that the related entities exist
        if (product.MetalId != 0)
        {
            var metalExists = await db.Metals.AnyAsync(m => m.Id == product.MetalId);
            if (!metalExists)
            {
                return Results.BadRequest(new { Message = $"Metal with ID {product.MetalId} not found" });
            }
        }

        if (product.CategoryId != 0)
        {
            var categoryExists = await db.Categories.AnyAsync(c => c.Id == product.CategoryId);
            if (!categoryExists)
            {
                return Results.BadRequest(new { Message = $"Category with ID {product.CategoryId} not found" });
            }
        }

        if (product.DiscountId.HasValue)
        {
            var discountExists = await db.Discounts.AnyAsync(d => d.Id == product.DiscountId.Value);
            if (!discountExists)
            {
                return Results.BadRequest(new { Message = $"Discount with ID {product.DiscountId.Value} not found" });
            }
        }

        // Save changes
        await db.SaveChangesAsync();

        return Results.Ok(product);
    }
    catch (Exception ex)
    {
        // Log the exception
        Console.WriteLine($"Error updating product: {ex.Message}");

        // Return a 500 Internal Server Error response
        return Results.Problem(
            title: "An error occurred while updating the product",
            detail: ex.Message,
            statusCode: 500
        );
    }
})
.WithName("PatchProduct")
.WithOpenApi();
```

To use this endpoint, you'll need to add the `Microsoft.AspNetCore.JsonPatch` package to your project:

```bash
dotnet add package Microsoft.AspNetCore.JsonPatch
```

And you'll need to add the necessary services to your application:

```csharp
builder.Services.AddControllers().AddNewtonsoftJson();
```

With this endpoint, clients can send a JSON Patch document to update specific fields of a product. For example:

```json
[
  { "op": "replace", "path": "/name", "value": "New Product Name" },
  { "op": "replace", "path": "/price", "value": 29.99 }
]
```

This would update only the name and price of the product, leaving other fields unchanged.

## Conclusion

In this chapter, you've learned how to implement endpoints for updating products in the Jewelry Junction API. You've explored different approaches to updates, including full updates and partial updates, and you've learned how to handle validation, related data, and response formatting.

In the next chapter, we'll implement the endpoint for joining related data, which will involve more complex queries and data manipulation.

## Practice Exercise

Enhance your product update functionality by:
1. Adding support for updating product images (URLs or file uploads)
2. Implementing versioning to track changes to products over time
3. Adding validation rules based on the product type (e.g., rings must have a size)
4. Implementing a feature to clone a product (create a new product based on an existing one)
5. Adding support for bulk updates (updating multiple products at once)