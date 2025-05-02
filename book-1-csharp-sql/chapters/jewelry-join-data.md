# Joining Related Data

In this chapter, we'll explore how to join related data in our Jewelry Junction API. We'll implement endpoints that retrieve data from multiple related tables, perform complex queries, and format the results in a meaningful way.

## Learning Objectives

By the end of this chapter, you should be able to:
- Understand the concept of joining related data in Entity Framework Core
- Implement endpoints that retrieve data from multiple related tables
- Use LINQ to perform complex queries
- Format joined data in a meaningful way
- Optimize queries for performance

## Understanding Joins in Entity Framework Core

Entity Framework Core provides several ways to join related data:
- **Eager loading**: Loading related data along with the main entity using the `Include` method
- **Explicit loading**: Loading related data explicitly after the main entity has been loaded
- **Lazy loading**: Loading related data automatically when a navigation property is accessed
- **Projection**: Selecting only the properties you need from multiple related entities

We've already used eager loading in previous chapters. In this chapter, we'll explore more complex joins and projections.

## Implementing a Product Details Endpoint

Let's start by implementing an endpoint that retrieves detailed information about a product, including its metal, category, discount, gemstones, and reviews:

```csharp
// Get product details
app.MapGet("/products/{id}/details", async (int id, JewelryContext db) =>
{
    try
    {
        // Find the product with all related data
        var product = await db.Products
            .AsNoTracking()
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
            return Results.NotFound(new { Message = $"Product with ID {id} not found" });
        }

        // Calculate average rating
        double averageRating = product.Reviews.Any()
            ? product.Reviews.Average(r => r.Rating)
            : 0;

        // Calculate total gemstone value
        decimal totalGemstoneValue = product.ProductGemstones.Sum(pg => pg.Carats * pg.Gemstone.PricePerCarat);

        // Calculate discounted price
        decimal? discountedPrice = null;
        if (product.Discount != null && product.Discount.IsActive)
        {
            discountedPrice = product.Price * (1 - product.Discount.DiscountPercent / 100);
        }

        // Format the response
        var response = new
        {
            product.Id,
            product.Name,
            product.Description,
            product.Price,
            DiscountedPrice = discountedPrice,
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
                Carats = pg.Carats,
                Value = pg.Carats * pg.Gemstone.PricePerCarat
            }).ToList(),
            TotalGemstoneValue = totalGemstoneValue,
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
            ReviewCount = product.Reviews.Count,
            TotalValue = product.Price + totalGemstoneValue
        };

        return Results.Ok(response);
    }
    catch (Exception ex)
    {
        // Log the exception
        Console.WriteLine($"Error retrieving product details: {ex.Message}");

        // Return a 500 Internal Server Error response
        return Results.Problem(
            title: "An error occurred while retrieving product details",
            detail: ex.Message,
            statusCode: 500
        );
    }
})
.WithName("GetProductDetails")
.WithOpenApi();
```

This endpoint:
1. Retrieves a product with all its related data using eager loading
2. Calculates derived values like average rating, total gemstone value, and discounted price
3. Formats the response to include all the relevant information in a structured way
## Implementing a Product Search Endpoint

Next, let's implement an endpoint that allows searching for products based on various criteria, including related data:

```csharp
// Search products
app.MapGet("/products/search", async (
    string? name,
    string? type,
    int? categoryId,
    int? metalId,
    int? gemstoneId,
    decimal? minPrice,
    decimal? maxPrice,
    bool? inStock,
    bool? discounted,
    int? minRating,
    JewelryContext db) =>
{
    try
    {
        // Start with a query that includes all products
        var query = db.Products
            .AsNoTracking()
            .Include(p => p.Metal)
            .Include(p => p.Category)
            .Include(p => p.Discount)
            .Include(p => p.ProductGemstones)
                .ThenInclude(pg => pg.Gemstone)
            .Include(p => p.Reviews)
            .AsQueryable();

        // Apply filters
        if (!string.IsNullOrEmpty(name))
        {
            query = query.Where(p => p.Name.Contains(name) || p.Description.Contains(name));
        }

        if (!string.IsNullOrEmpty(type))
        {
            query = query.Where(p => p.Type == type);
        }

        if (categoryId.HasValue)
        {
            query = query.Where(p => p.CategoryId == categoryId.Value);
        }

        if (metalId.HasValue)
        {
            query = query.Where(p => p.MetalId == metalId.Value);
        }

        if (gemstoneId.HasValue)
        {
            query = query.Where(p => p.ProductGemstones.Any(pg => pg.GemstoneId == gemstoneId.Value));
        }

        if (minPrice.HasValue)
        {
            query = query.Where(p => p.Price >= minPrice.Value);
        }

        if (maxPrice.HasValue)
        {
            query = query.Where(p => p.Price <= maxPrice.Value);
        }

        if (inStock.HasValue)
        {
            query = query.Where(p => inStock.Value ? p.StockQuantity > 0 : p.StockQuantity == 0);
        }

        if (discounted.HasValue)
        {
            query = query.Where(p => discounted.Value
                ? p.DiscountId != null && p.Discount.IsActive
                : p.DiscountId == null || !p.Discount.IsActive);
        }

        if (minRating.HasValue)
        {
            query = query.Where(p => p.Reviews.Any() && p.Reviews.Average(r => r.Rating) >= minRating.Value);
        }

        // Execute the query
        var products = await query.ToListAsync();

        // Format the response
        var response = products.Select(p =>
        {
            // Calculate average rating
            double averageRating = p.Reviews.Any()
                ? p.Reviews.Average(r => r.Rating)
                : 0;

            // Calculate discounted price
            decimal? discountedPrice = null;
            if (p.Discount != null && p.Discount.IsActive)
            {
                discountedPrice = p.Price * (1 - p.Discount.DiscountPercent / 100);
            }

            return new
            {
                p.Id,
                p.Name,
                p.Description,
                p.Price,
                DiscountedPrice = discountedPrice,
                p.StockQuantity,
                p.Type,
                Metal = new
                {
                    p.Metal.Id,
                    p.Metal.Name
                },
                Category = new
                {
                    p.Category.Id,
                    p.Category.Name
                },
                Discount = p.Discount != null && p.Discount.IsActive ? new
                {
                    p.Discount.Id,
                    p.Discount.Name,
                    p.Discount.DiscountPercent
                } : null,
                Gemstones = p.ProductGemstones.Select(pg => new
                {
                    Id = pg.Gemstone.Id,
                    Name = pg.Gemstone.Name,
                    Carats = pg.Carats
                }).ToList(),
                AverageRating = averageRating,
                ReviewCount = p.Reviews.Count,
                IsInStock = p.StockQuantity > 0,
                IsDiscounted = p.Discount != null && p.Discount.IsActive
            };
        }).ToList();

        return Results.Ok(new
        {
            Products = response,
            Count = response.Count
        });
    }
    catch (Exception ex)
    {
        // Log the exception
        Console.WriteLine($"Error searching products: {ex.Message}");

        // Return a 500 Internal Server Error response
        return Results.Problem(
            title: "An error occurred while searching products",
            detail: ex.Message,
            statusCode: 500
        );
    }
})
.WithName("SearchProducts")
.WithOpenApi();
```

This endpoint:
1. Allows searching for products based on various criteria, including related data
2. Builds a query dynamically based on the provided search parameters
3. Formats the response to include relevant information about each product

## Implementing a Sales Report Endpoint

Now, let's implement an endpoint that generates a sales report by joining order data with product data:

```csharp
// Get sales report
app.MapGet("/reports/sales", async (
    DateTime? startDate,
    DateTime? endDate,
    string? groupBy,
    JewelryContext db) =>
{
    try
    {
        // Set default dates if not provided
        startDate ??= DateTime.Now.AddMonths(-1);
        endDate ??= DateTime.Now;

        // Validate dates
        if (startDate > endDate)
        {
            return Results.BadRequest(new { Message = "Start date cannot be after end date" });
        }

        // Get all orders within the date range
        var orders = await db.Orders
            .AsNoTracking()
            .Where(o => o.OrderDate >= startDate && o.OrderDate <= endDate && !o.IsDeleted)
            .Include(o => o.OrderItems)
                .ThenInclude(oi => oi.Product)
                    .ThenInclude(p => p.Category)
            .Include(o => o.OrderItems)
                .ThenInclude(oi => oi.Product)
                    .ThenInclude(p => p.Metal)
            .ToListAsync();

        // Calculate total sales
        decimal totalSales = orders.Sum(o => o.TotalAmount);

        // Calculate total items sold
        int totalItemsSold = orders.Sum(o => o.OrderItems.Sum(oi => oi.Quantity));

        // Calculate average order value
        decimal averageOrderValue = orders.Any() ? totalSales / orders.Count : 0;

        // Group sales data based on the groupBy parameter
        object salesByGroup;

        switch (groupBy?.ToLower())
        {
            case "category":
                salesByGroup = orders
                    .SelectMany(o => o.OrderItems)
                    .GroupBy(oi => oi.Product.Category.Name)
                    .Select(g => new
                    {
                        Category = g.Key,
                        TotalSales = g.Sum(oi => oi.UnitPrice * oi.Quantity),
                        ItemsSold = g.Sum(oi => oi.Quantity),
                        AveragePrice = g.Sum(oi => oi.UnitPrice * oi.Quantity) / g.Sum(oi => oi.Quantity)
                    })
                    .OrderByDescending(x => x.TotalSales)
                    .ToList();
                break;

            case "metal":
                salesByGroup = orders
                    .SelectMany(o => o.OrderItems)
                    .GroupBy(oi => oi.Product.Metal.Name)
                    .Select(g => new
                    {
                        Metal = g.Key,
                        TotalSales = g.Sum(oi => oi.UnitPrice * oi.Quantity),
                        ItemsSold = g.Sum(oi => oi.Quantity),
                        AveragePrice = g.Sum(oi => oi.UnitPrice * oi.Quantity) / g.Sum(oi => oi.Quantity)
                    })
                    .OrderByDescending(x => x.TotalSales)
                    .ToList();
                break;

            case "product":
                salesByGroup = orders
                    .SelectMany(o => o.OrderItems)
                    .GroupBy(oi => new { oi.Product.Id, oi.Product.Name })
                    .Select(g => new
                    {
                        ProductId = g.Key.Id,
                        ProductName = g.Key.Name,
                        TotalSales = g.Sum(oi => oi.UnitPrice * oi.Quantity),
                        ItemsSold = g.Sum(oi => oi.Quantity),
                        AveragePrice = g.Sum(oi => oi.UnitPrice * oi.Quantity) / g.Sum(oi => oi.Quantity)
                    })
                    .OrderByDescending(x => x.TotalSales)
                    .ToList();
                break;

            case "date":
                salesByGroup = orders
                    .GroupBy(o => o.OrderDate.Date)
                    .Select(g => new
                    {
                        Date = g.Key.ToString("yyyy-MM-dd"),
                        TotalSales = g.Sum(o => o.TotalAmount),
                        OrderCount = g.Count(),
                        AverageOrderValue = g.Sum(o => o.TotalAmount) / g.Count()
                    })
                    .OrderBy(x => x.Date)
                    .ToList();
                break;

            default:
                salesByGroup = new { Message = "No grouping specified. Use 'category', 'metal', 'product', or 'date'." };
                break;
        }

        // Format the response
        var response = new
        {
            StartDate = startDate.Value.ToString("yyyy-MM-dd"),
            EndDate = endDate.Value.ToString("yyyy-MM-dd"),
            TotalSales = totalSales,
            TotalItemsSold = totalItemsSold,
            AverageOrderValue = averageOrderValue,
            OrderCount = orders.Count,
            GroupedBy = groupBy ?? "none",
            SalesByGroup = salesByGroup
        };

        return Results.Ok(response);
    }
    catch (Exception ex)
    {
        // Log the exception
        Console.WriteLine($"Error generating sales report: {ex.Message}");
## Implementing a Customer Order History Endpoint

Let's implement an endpoint that retrieves a customer's order history, including all the products they've ordered:

```csharp
// Get customer order history
app.MapGet("/customers/{id}/orders", async (int id, JewelryContext db) =>
{
    try
    {
        // Find the customer
        var customer = await db.Customers
            .AsNoTracking()
            .FirstOrDefaultAsync(c => c.Id == id);

        if (customer == null)
        {
            return Results.NotFound(new { Message = $"Customer with ID {id} not found" });
        }

        // Get the customer's orders
        var orders = await db.Orders
            .AsNoTracking()
            .Where(o => o.CustomerId == id && !o.IsDeleted)
            .Include(o => o.OrderItems)
                .ThenInclude(oi => oi.Product)
                    .ThenInclude(p => p.Metal)
            .Include(o => o.OrderItems)
                .ThenInclude(oi => oi.Product)
                    .ThenInclude(p => p.Category)
            .OrderByDescending(o => o.OrderDate)
            .ToListAsync();

        // Format the response
        var response = new
        {
            Customer = new
            {
                customer.Id,
                Name = $"{customer.FirstName} {customer.LastName}",
                customer.Email,
                customer.Phone,
                customer.Address
            },
            Orders = orders.Select(o => new
            {
                o.Id,
                o.OrderDate,
                o.Status,
                o.TotalAmount,
                Items = o.OrderItems.Select(oi => new
                {
                    oi.Id,
                    Product = new
                    {
                        oi.Product.Id,
                        oi.Product.Name,
                        oi.Product.Type,
                        Metal = oi.Product.Metal.Name,
                        Category = oi.Product.Category.Name
                    },
                    oi.Quantity,
                    oi.UnitPrice,
                    Subtotal = oi.Quantity * oi.UnitPrice
                }).ToList(),
                ItemCount = o.OrderItems.Count,
                TotalItems = o.OrderItems.Sum(oi => oi.Quantity)
            }).ToList(),
            OrderCount = orders.Count,
            TotalSpent = orders.Sum(o => o.TotalAmount),
            AverageOrderValue = orders.Any() ? orders.Average(o => o.TotalAmount) : 0
        };

        return Results.Ok(response);
    }
    catch (Exception ex)
    {
        // Log the exception
        Console.WriteLine($"Error retrieving customer order history: {ex.Message}");

        // Return a 500 Internal Server Error response
        return Results.Problem(
            title: "An error occurred while retrieving customer order history",
            detail: ex.Message,
            statusCode: 500
        );
    }
})
.WithName("GetCustomerOrderHistory")
.WithOpenApi();
```

This endpoint:
1. Retrieves a customer's orders, including all the products they've ordered
2. Formats the response to include customer information, order details, and summary metrics

## Implementing a Product Recommendations Endpoint

Finally, let's implement an endpoint that provides product recommendations based on a customer's order history:

```csharp
// Get product recommendations for a customer
app.MapGet("/customers/{id}/recommendations", async (int id, JewelryContext db) =>
{
    try
    {
        // Find the customer
        var customer = await db.Customers
            .AsNoTracking()
            .FirstOrDefaultAsync(c => c.Id == id);

        if (customer == null)
        {
            return Results.NotFound(new { Message = $"Customer with ID {id} not found" });
        }

        // Get the customer's orders
        var customerOrders = await db.Orders
            .AsNoTracking()
            .Where(o => o.CustomerId == id && !o.IsDeleted)
            .Include(o => o.OrderItems)
            .ToListAsync();

        // Get the products the customer has ordered
        var orderedProductIds = customerOrders
            .SelectMany(o => o.OrderItems)
            .Select(oi => oi.ProductId)
            .Distinct()
            .ToList();

        // Get the categories and metals the customer has shown interest in
        var orderedProducts = await db.Products
            .AsNoTracking()
            .Where(p => orderedProductIds.Contains(p.Id))
            .ToListAsync();

        var preferredCategoryIds = orderedProducts
            .Select(p => p.CategoryId)
            .Distinct()
            .ToList();

        var preferredMetalIds = orderedProducts
            .Select(p => p.MetalId)
            .Distinct()
            .ToList();

        // Get recommendations based on categories and metals the customer has shown interest in
        var recommendations = await db.Products
            .AsNoTracking()
            .Where(p => !orderedProductIds.Contains(p.Id) && // Exclude products the customer has already ordered
                       (preferredCategoryIds.Contains(p.CategoryId) || preferredMetalIds.Contains(p.MetalId)) &&
                       p.StockQuantity > 0) // Only recommend products that are in stock
            .Include(p => p.Metal)
            .Include(p => p.Category)
            .Include(p => p.Discount)
            .Include(p => p.Reviews)
            .OrderByDescending(p => p.Reviews.Any() ? p.Reviews.Average(r => r.Rating) : 0) // Order by rating
            .Take(10) // Limit to 10 recommendations
            .ToListAsync();

        // Format the response
        var response = new
        {
            Customer = new
            {
                customer.Id,
                Name = $"{customer.FirstName} {customer.LastName}",
                customer.Email
            },
            Recommendations = recommendations.Select(p =>
            {
                // Calculate average rating
                double averageRating = p.Reviews.Any()
                    ? p.Reviews.Average(r => r.Rating)
                    : 0;

                // Calculate discounted price
                decimal? discountedPrice = null;
                if (p.Discount != null && p.Discount.IsActive)
                {
                    discountedPrice = p.Price * (1 - p.Discount.DiscountPercent / 100);
                }

                return new
                {
                    p.Id,
                    p.Name,
                    p.Description,
                    p.Price,
                    DiscountedPrice = discountedPrice,
                    p.Type,
                    Metal = new
                    {
                        p.Metal.Id,
                        p.Metal.Name
                    },
                    Category = new
                    {
                        p.Category.Id,
                        p.Category.Name
                    },
                    AverageRating = averageRating,
                    ReviewCount = p.Reviews.Count,
                    IsDiscounted = p.Discount != null && p.Discount.IsActive,
                    RecommendationReason = preferredCategoryIds.Contains(p.CategoryId) && preferredMetalIds.Contains(p.MetalId)
                        ? "Based on your interest in both this category and metal"
                        : preferredCategoryIds.Contains(p.CategoryId)
                            ? "Based on your interest in this category"
                            : "Based on your interest in this metal"
                };
            }).ToList(),
            RecommendationCount = recommendations.Count
        };

        return Results.Ok(response);
    }
    catch (Exception ex)
    {
        // Log the exception
        Console.WriteLine($"Error generating product recommendations: {ex.Message}");

        // Return a 500 Internal Server Error response
        return Results.Problem(
            title: "An error occurred while generating product recommendations",
            detail: ex.Message,
            statusCode: 500
        );
    }
})
.WithName("GetProductRecommendations")
.WithOpenApi();
```

This endpoint:
1. Analyzes a customer's order history to determine their preferences
2. Recommends products based on categories and metals the customer has shown interest in
3. Excludes products the customer has already ordered
4. Orders recommendations by rating and limits the results to 10 products
5. Includes a reason for each recommendation

## Optimizing Queries

When working with complex joins and queries, it's important to optimize them for performance. Here are some techniques to optimize your queries:

1. **Use projection**: Select only the properties you need instead of loading entire entities
2. **Use `AsNoTracking`**: When you're only reading data and not modifying it, use `AsNoTracking` to improve performance
3. **Filter data early**: Apply filters as early as possible in the query to reduce the amount of data processed
4. **Use paging**: When retrieving large result sets, use paging to limit the amount of data returned
5. **Use appropriate indexes**: Ensure that your database has appropriate indexes for the columns you're querying
6. **Avoid N+1 queries**: Use eager loading with `Include` to avoid multiple database roundtrips

Let's update our product search endpoint to use projection and paging:

```csharp
// Search products with projection and paging
app.MapGet("/products/search", async (
    string? name,
    string? type,
    int? categoryId,
    int? metalId,
    int? gemstoneId,
    decimal? minPrice,
    decimal? maxPrice,
    bool? inStock,
    bool? discounted,
    int? minRating,
    int page = 1,
    int pageSize = 10,
    JewelryContext db) =>
{
    try
    {
        // Start with a query that includes all products
        var query = db.Products
            .AsNoTracking()
            .AsQueryable();

        // Apply filters
        if (!string.IsNullOrEmpty(name))
        {
            query = query.Where(p => p.Name.Contains(name) || p.Description.Contains(name));
        }

        if (!string.IsNullOrEmpty(type))
        {
            query = query.Where(p => p.Type == type);
        }

        if (categoryId.HasValue)
        {
            query = query.Where(p => p.CategoryId == categoryId.Value);
        }

        if (metalId.HasValue)
        {
            query = query.Where(p => p.MetalId == metalId.Value);
        }

        if (gemstoneId.HasValue)
        {
            query = query.Where(p => p.ProductGemstones.Any(pg => pg.GemstoneId == gemstoneId.Value));
        }

        if (minPrice.HasValue)
        {
            query = query.Where(p => p.Price >= minPrice.Value);
        }

        if (maxPrice.HasValue)
        {
            query = query.Where(p => p.Price <= maxPrice.Value);
        }

        if (inStock.HasValue)
        {
            query = query.Where(p => inStock.Value ? p.StockQuantity > 0 : p.StockQuantity == 0);
        }

        if (discounted.HasValue)
        {
            query = query.Where(p => discounted.Value
                ? p.DiscountId != null && p.Discount.IsActive
                : p.DiscountId == null || !p.Discount.IsActive);
        }

        if (minRating.HasValue)
        {
            query = query.Where(p => p.Reviews.Any() && p.Reviews.Average(r => r.Rating) >= minRating.Value);
        }

        // Get total count for pagination
        var totalCount = await query.CountAsync();

        // Apply paging
        var pagedQuery = query
            .Skip((page - 1) * pageSize)
            .Take(pageSize);

        // Use projection to select only the properties we need
        var products = await pagedQuery
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
                    Name = p.Metal.Name
                },
                Category = new
                {
                    Id = p.Category.Id,
                    Name = p.Category.Name
                },
                Discount = p.Discount != null && p.Discount.IsActive ? new
                {
                    Id = p.Discount.Id,
                    Name = p.Discount.Name,
                    DiscountPercent = p.Discount.DiscountPercent
                } : null,
                GemstoneCount = p.ProductGemstones.Count,
                AverageRating = p.Reviews.Any() ? p.Reviews.Average(r => r.Rating) : 0,
                ReviewCount = p.Reviews.Count,
                IsInStock = p.StockQuantity > 0,
                IsDiscounted = p.Discount != null && p.Discount.IsActive
            })
            .ToListAsync();

        // Create pagination metadata
        var pagination = new
        {
            TotalCount = totalCount,
            TotalPages = (int)Math.Ceiling(totalCount / (double)pageSize),
            CurrentPage = page,
            PageSize = pageSize,
            HasPrevious = page > 1,
            HasNext = page < (int)Math.Ceiling(totalCount / (double)pageSize)
        };

        return Results.Ok(new
        {
            Products = products,
            Pagination = pagination
        });
    }
    catch (Exception ex)
    {
        // Log the exception
        Console.WriteLine($"Error searching products: {ex.Message}");

        // Return a 500 Internal Server Error response
        return Results.Problem(
            title: "An error occurred while searching products",
            detail: ex.Message,
            statusCode: 500
        );
    }
})
.WithName("SearchProducts")
.WithOpenApi();
```

This optimized endpoint:
1. Uses projection to select only the properties we need
2. Applies paging to limit the amount of data returned
3. Includes pagination metadata in the response

## Conclusion

In this chapter, you've learned how to join related data in the Jewelry Junction API. You've implemented endpoints that retrieve data from multiple related tables, perform complex queries, and format the results in a meaningful way. You've also learned how to optimize queries for performance.

These techniques are essential for building robust and efficient APIs that can handle complex data relationships and provide meaningful insights to clients.

## Practice Exercise

Enhance your Jewelry Junction API by:
1. Implementing an endpoint that retrieves the top-selling products for a given time period
2. Creating an endpoint that provides inventory status reports (e.g., products with low stock, out-of-stock products)
3. Implementing an endpoint that calculates the total value of the inventory
4. Creating an endpoint that provides customer insights (e.g., top customers, customer purchase patterns)
5. Implementing an endpoint that generates a report of products that haven't sold in a given time period

        // Return a 500 Internal Server Error response
        return Results.Problem(
            title: "An error occurred while generating the sales report",
            detail: ex.Message,
            statusCode: 500
        );
    }
})
.WithName("GetSalesReport")
.WithOpenApi();
```

This endpoint:
1. Retrieves orders within a specified date range
2. Joins order data with product data to calculate sales metrics
3. Groups the data based on the provided `groupBy` parameter (category, metal, product, or date)
4. Formats the response to include summary metrics and grouped data