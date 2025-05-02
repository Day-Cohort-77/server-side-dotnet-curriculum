# Get Single Product

In this chapter, we'll implement and enhance the endpoint for retrieving a single product by ID in our Jewelry Junction API. We'll explore how to include related data, handle errors, and format the response.

## Learning Objectives

By the end of this chapter, you should be able to:
- Implement a GET endpoint to retrieve a single product by ID
- Include related data in the response
- Handle cases where the product doesn't exist
- Format the response data
- Implement error handling

## Implementing the Basic Endpoint

We've already implemented a basic endpoint for retrieving a single product by ID in a previous chapter. Let's review and enhance it:

```csharp
// Get product by ID
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
```

This endpoint retrieves a single product by ID, including related data such as metal, category, discount, gemstones, and reviews. It returns a 404 Not Found response if the product doesn't exist.

Let's enhance this endpoint to provide a more structured and informative response.

## Formatting the Response

First, let's format the response to include only the necessary information and to structure it in a more user-friendly way:

```csharp
// Get product by ID with formatted response
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

    if (product == null)
    {
        return Results.NotFound();
    }

    // Calculate average rating
    double averageRating = product.Reviews.Any()
        ? product.Reviews.Average(r => r.Rating)
        : 0;

    // Format the response
    var formattedProduct = new
    {
        product.Id,
        product.Name,
        product.Description,
        product.Price,
        DiscountedPrice = product.Discount != null && product.Discount.IsActive
            ? product.Price * (1 - product.Discount.DiscountPercent / 100)
            : (decimal?)null,
        product.StockQuantity,
        product.Type,
        Metal = new
        {
            product.Metal.Id,
            product.Metal.Name,
            product.Metal.PricePerGram
        },
        Category = new
        {
            product.Category.Id,
            product.Category.Name,
            product.Category.Description
        },
        Discount = product.Discount != null ? new
        {
            product.Discount.Id,
            product.Discount.Name,
            product.Discount.Description,
            product.Discount.DiscountPercent,
            product.Discount.StartDate,
            product.Discount.EndDate,
            product.Discount.IsActive
        } : null,
        Gemstones = product.ProductGemstones.Select(pg => new
        {
            Id = pg.Gemstone.Id,
            Name = pg.Gemstone.Name,
            PricePerCarat = pg.Gemstone.PricePerCarat,
            Carats = pg.Carats
        }).ToList(),
        Reviews = product.Reviews.Select(r => new
        {
            r.Id,
            r.Rating,
            r.Comment,
            r.ReviewDate,
            Customer = new
            {
                r.Customer.Id,
                Name = $"{r.Customer.FirstName} {r.Customer.LastName}",
                r.Customer.Email
            }
        }).ToList(),
        AverageRating = averageRating,
        ReviewCount = product.Reviews.Count
    };

    return Results.Ok(formattedProduct);
})
.WithName("GetProductById")
.WithOpenApi();
```

Now, the response includes a well-structured representation of the product with all its related data, as well as calculated fields like average rating and review count.

## Optimizing the Query

To optimize the query, we can use projection to select only the properties we need, rather than loading entire entities:

```csharp
// Get product by ID with optimized query
app.MapGet("/products/{id}", async (int id, JewelryContext db) =>
{
    // Check if the product exists
    var productExists = await db.Products.AnyAsync(p => p.Id == id);

    if (!productExists)
    {
        return Results.NotFound();
    }

    // Retrieve the product with projection
    var product = await db.Products
        .AsNoTracking() // Improves performance for read-only queries
        .Where(p => p.Id == id)
        .Select(p => new
        {
            p.Id,
            p.Name,
            p.Description,
            p.Price,
            DiscountedPrice = p.Discount != null && p.Discount.IsActive
                ? p.Price * (1 - p.Discount.DiscountPercent / 100)
                : (decimal?)null,
            p.StockQuantity,
            p.Type,
            Metal = new
            {
                Id = p.Metal.Id,
                Name = p.Metal.Name,
                PricePerGram = p.Metal.PricePerGram
            },
            Category = new
            {
                Id = p.Category.Id,
                Name = p.Category.Name,
                Description = p.Category.Description
            },
            Discount = p.Discount != null ? new
            {
                Id = p.Discount.Id,
                Name = p.Discount.Name,
                Description = p.Discount.Description,
                DiscountPercent = p.Discount.DiscountPercent,
                StartDate = p.Discount.StartDate,
                EndDate = p.Discount.EndDate,
                IsActive = p.Discount.IsActive
            } : null,
            Gemstones = p.ProductGemstones.Select(pg => new
            {
                Id = pg.Gemstone.Id,
                Name = pg.Gemstone.Name,
                PricePerCarat = pg.Gemstone.PricePerCarat,
                Carats = pg.Carats
            }).ToList(),
            Reviews = p.Reviews.Select(r => new
            {
                r.Id,
                r.Rating,
                r.Comment,
                r.ReviewDate,
                Customer = new
                {
                    r.Customer.Id,
                    Name = $"{r.Customer.FirstName} {r.Customer.LastName}",
                    r.Customer.Email
                }
            }).ToList(),
            AverageRating = p.Reviews.Any() ? p.Reviews.Average(r => r.Rating) : 0,
            ReviewCount = p.Reviews.Count
        })
        .FirstOrDefaultAsync();

    return Results.Ok(product);
})
.WithName("GetProductById")
.WithOpenApi();
```

In this optimized version:
- We first check if the product exists using a simple query
- We use `AsNoTracking()` to improve performance for read-only queries
- We use projection with `Select()` to retrieve only the properties we need, rather than loading entire entities
- We perform the projection in the database query, rather than in memory

## Adding Error Handling

Let's add more robust error handling to our endpoint:

```csharp
// Get product by ID with error handling
app.MapGet("/products/{id}", async (int id, JewelryContext db) =>
{
    try
    {
        // Check if the product exists
        var productExists = await db.Products.AnyAsync(p => p.Id == id);

        if (!productExists)
        {
            return Results.NotFound(new { Message = $"Product with ID {id} not found" });
        }

        // Retrieve the product with projection
        var product = await db.Products
            .AsNoTracking()
            .Where(p => p.Id == id)
            .Select(p => new
            {
                p.Id,
                p.Name,
                p.Description,
                p.Price,
                DiscountedPrice = p.Discount != null && p.Discount.IsActive
                    ? p.Price * (1 - p.Discount.DiscountPercent / 100)
                    : (decimal?)null,
                p.StockQuantity,
                p.Type,
                Metal = new
                {
                    Id = p.Metal.Id,
                    Name = p.Metal.Name,
                    PricePerGram = p.Metal.PricePerGram
                },
                Category = new
                {
                    Id = p.Category.Id,
                    Name = p.Category.Name,
                    Description = p.Category.Description
                },
                Discount = p.Discount != null ? new
                {
                    Id = p.Discount.Id,
                    Name = p.Discount.Name,
                    Description = p.Discount.Description,
                    DiscountPercent = p.Discount.DiscountPercent,
                    StartDate = p.Discount.StartDate,
                    EndDate = p.Discount.EndDate,
                    IsActive = p.Discount.IsActive
                } : null,
                Gemstones = p.ProductGemstones.Select(pg => new
                {
                    Id = pg.Gemstone.Id,
                    Name = pg.Gemstone.Name,
                    PricePerCarat = pg.Gemstone.PricePerCarat,
                    Carats = pg.Carats
                }).ToList(),
                Reviews = p.Reviews.Select(r => new
                {
                    r.Id,
                    r.Rating,
                    r.Comment,
                    r.ReviewDate,
                    Customer = new
                    {
                        r.Customer.Id,
                        Name = $"{r.Customer.FirstName} {r.Customer.LastName}",
                        r.Customer.Email
                    }
                }).ToList(),
                AverageRating = p.Reviews.Any() ? p.Reviews.Average(r => r.Rating) : 0,
                ReviewCount = p.Reviews.Count
            })
            .FirstOrDefaultAsync();

        return Results.Ok(product);
    }
    catch (Exception ex)
    {
        // Log the exception
        Console.WriteLine($"Error retrieving product: {ex.Message}");

        // Return a 500 Internal Server Error response
        return Results.Problem(
            title: "An error occurred while retrieving the product",
            detail: ex.Message,
            statusCode: 500
        );
    }
})
.WithName("GetProductById")
.WithOpenApi();
```

Now, the endpoint includes proper error handling:
- It returns a 404 Not Found response with a descriptive message if the product doesn't exist
- It catches any exceptions that might occur during the query and returns a 500 Internal Server Error response with details about the error

## Adding Related Endpoints

Let's add some related endpoints to provide more ways to access product information:

### Get Product Reviews

```csharp
// Get product reviews
app.MapGet("/products/{id}/reviews", async (int id, JewelryContext db) =>
{
    try
    {
        // Check if the product exists
        var productExists = await db.Products.AnyAsync(p => p.Id == id);

        if (!productExists)
        {
            return Results.NotFound(new { Message = $"Product with ID {id} not found" });
        }

        // Retrieve the product reviews
        var reviews = await db.Reviews
            .AsNoTracking()
            .Where(r => r.ProductId == id)
            .Select(r => new
            {
                r.Id,
                r.Rating,
                r.Comment,
                r.ReviewDate,
                Customer = new
                {
                    r.Customer.Id,
                    Name = $"{r.Customer.FirstName} {r.Customer.LastName}",
                    r.Customer.Email
                }
            })
            .ToListAsync();

        // Calculate average rating
        double averageRating = reviews.Any()
            ? reviews.Average(r => r.Rating)
            : 0;

        return Results.Ok(new
        {
            Reviews = reviews,
            AverageRating = averageRating,
            ReviewCount = reviews.Count
        });
    }
    catch (Exception ex)
    {
        // Log the exception
        Console.WriteLine($"Error retrieving product reviews: {ex.Message}");

        // Return a 500 Internal Server Error response
        return Results.Problem(
            title: "An error occurred while retrieving the product reviews",
            detail: ex.Message,
            statusCode: 500
        );
    }
})
.WithName("GetProductReviews")
.WithOpenApi();
```

### Get Product Gemstones

```csharp
// Get product gemstones
app.MapGet("/products/{id}/gemstones", async (int id, JewelryContext db) =>
{
    try
    {
        // Check if the product exists
        var productExists = await db.Products.AnyAsync(p => p.Id == id);

        if (!productExists)
        {
            return Results.NotFound(new { Message = $"Product with ID {id} not found" });
        }

        // Retrieve the product gemstones
        var gemstones = await db.ProductGemstones
            .AsNoTracking()
            .Where(pg => pg.ProductId == id)
            .Select(pg => new
            {
                Id = pg.Gemstone.Id,
                Name = pg.Gemstone.Name,
                PricePerCarat = pg.Gemstone.PricePerCarat,
                Carats = pg.Carats,
                TotalValue = pg.Carats * pg.Gemstone.PricePerCarat
            })
            .ToListAsync();

        return Results.Ok(gemstones);
    }
    catch (Exception ex)
    {
        // Log the exception
        Console.WriteLine($"Error retrieving product gemstones: {ex.Message}");

        // Return a 500 Internal Server Error response
        return Results.Problem(
            title: "An error occurred while retrieving the product gemstones",
            detail: ex.Message,
            statusCode: 500
        );
    }
})
.WithName("GetProductGemstones")
.WithOpenApi();
```

### Check Product Availability

```csharp
// Check product availability
app.MapGet("/products/{id}/availability", async (int id, JewelryContext db) =>
{
    try
    {
        // Check if the product exists and retrieve its stock quantity
        var product = await db.Products
            .AsNoTracking()
            .Where(p => p.Id == id)
            .Select(p => new { p.Id, p.Name, p.StockQuantity })
            .FirstOrDefaultAsync();

        if (product == null)
        {
            return Results.NotFound(new { Message = $"Product with ID {id} not found" });
        }

        // Determine availability status
        string status;
        if (product.StockQuantity > 10)
        {
            status = "In Stock";
        }
        else if (product.StockQuantity > 0)
        {
            status = "Low Stock";
        }
        else
        {
            status = "Out of Stock";
        }

        return Results.Ok(new
        {
            product.Id,
            product.Name,
            product.StockQuantity,
            Status = status,
            IsAvailable = product.StockQuantity > 0
        });
    }
    catch (Exception ex)
    {
        // Log the exception
        Console.WriteLine($"Error checking product availability: {ex.Message}");

        // Return a 500 Internal Server Error response
        return Results.Problem(
            title: "An error occurred while checking product availability",
            detail: ex.Message,
            statusCode: 500
        );
    }
})
.WithName("CheckProductAvailability")
.WithOpenApi();
```

## Conclusion

In this chapter, you've learned how to implement and enhance the endpoint for retrieving a single product by ID in the Jewelry Junction API. You've included related data in the response, handled cases where the product doesn't exist, formatted the response data, and implemented error handling. You've also added related endpoints to provide more ways to access product information.

In the next chapter, we'll implement the endpoint for creating a new order, which will involve more complex operations like transactions and validation.

## Practice Exercise

Enhance your Get Single Product endpoint by:
1. Adding a parameter to control the inclusion of related data (e.g., `?includeReviews=true&includeGemstones=true`)
2. Adding a parameter to specify the number of reviews to include (e.g., `?reviewCount=5` for the 5 most recent reviews)
3. Adding a calculated field for the total value of the product (base price + gemstone value)
4. Adding a field that indicates whether the product is on sale (has an active discount)
5. Creating a new endpoint that returns similar products (products in the same category or with the same metal)