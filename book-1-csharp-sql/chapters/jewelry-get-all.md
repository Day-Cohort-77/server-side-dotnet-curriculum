# Get All Products

In this chapter, we'll implement and enhance the endpoint for retrieving all products in our Jewelry Junction API. We'll explore how to optimize queries, implement filtering, sorting, and pagination, and format the response data.

## Learning Objectives

By the end of this chapter, you should be able to:
- Implement a GET endpoint to retrieve all products
- Add filtering capabilities to the endpoint
- Implement sorting options
- Add pagination to handle large result sets
- Format and shape the response data
- Optimize database queries for performance

## Implementing the Basic Endpoint

We've already implemented a basic endpoint for retrieving all products in the previous chapter. Let's review and enhance it:

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
```

This endpoint retrieves all products from the database, including related metal, category, and discount data. However, it has several limitations:
- It returns all products without any filtering
- There's no sorting capability
- It doesn't handle pagination, which could be problematic with large datasets
- The response format might include more data than needed

Let's address these limitations one by one.

## Adding Filtering Capabilities

First, let's add filtering capabilities to our endpoint. We'll allow filtering by category, metal, price range, and whether the product is discounted:

```csharp
// Get all products with filtering
app.MapGet("/products", async (
    JewelryContext db,
    int? categoryId,
    int? metalId,
    decimal? minPrice,
    decimal? maxPrice,
    bool? discounted) =>
{
    var query = db.Products
        .Include(p => p.Metal)
        .Include(p => p.Category)
        .Include(p => p.Discount)
        .AsQueryable();

    // Apply filters
    if (categoryId.HasValue)
    {
        query = query.Where(p => p.CategoryId == categoryId.Value);
    }

    if (metalId.HasValue)
    {
        query = query.Where(p => p.MetalId == metalId.Value);
    }

    if (minPrice.HasValue)
    {
        query = query.Where(p => p.Price >= minPrice.Value);
    }

    if (maxPrice.HasValue)
    {
        query = query.Where(p => p.Price <= maxPrice.Value);
    }

    if (discounted.HasValue && discounted.Value)
    {
        query = query.Where(p => p.DiscountId != null && p.Discount.IsActive);
    }

    var products = await query.ToListAsync();
    return Results.Ok(products);
})
.WithName("GetAllProducts")
.WithOpenApi();
```

Now, users can filter products by adding query parameters to the URL, such as:
- `/products?categoryId=1` - Get all products in category 1
- `/products?minPrice=100&maxPrice=500` - Get all products with a price between $100 and $500
- `/products?discounted=true` - Get all products with an active discount

## Implementing Sorting

Next, let's add sorting capabilities to our endpoint. We'll allow sorting by name, price, and stock quantity:

```csharp
// Get all products with filtering and sorting
app.MapGet("/products", async (
    JewelryContext db,
    int? categoryId,
    int? metalId,
    decimal? minPrice,
    decimal? maxPrice,
    bool? discounted,
    string? sortBy,
    bool? sortDesc) =>
{
    var query = db.Products
        .Include(p => p.Metal)
        .Include(p => p.Category)
        .Include(p => p.Discount)
        .AsQueryable();

    // Apply filters
    if (categoryId.HasValue)
    {
        query = query.Where(p => p.CategoryId == categoryId.Value);
    }

    if (metalId.HasValue)
    {
        query = query.Where(p => p.MetalId == metalId.Value);
    }

    if (minPrice.HasValue)
    {
        query = query.Where(p => p.Price >= minPrice.Value);
    }

    if (maxPrice.HasValue)
    {
        query = query.Where(p => p.Price <= maxPrice.Value);
    }

    if (discounted.HasValue && discounted.Value)
    {
        query = query.Where(p => p.DiscountId != null && p.Discount.IsActive);
    }

    // Apply sorting
    if (!string.IsNullOrEmpty(sortBy))
    {
        bool isDescending = sortDesc ?? false;

        query = sortBy.ToLower() switch
        {
            "name" => isDescending
                ? query.OrderByDescending(p => p.Name)
                : query.OrderBy(p => p.Name),
            "price" => isDescending
                ? query.OrderByDescending(p => p.Price)
                : query.OrderBy(p => p.Price),
            "stock" => isDescending
                ? query.OrderByDescending(p => p.StockQuantity)
                : query.OrderBy(p => p.StockQuantity),
            _ => query.OrderBy(p => p.Id) // Default sorting
        };
    }
    else
    {
        // Default sorting by ID
        query = query.OrderBy(p => p.Id);
    }

    var products = await query.ToListAsync();
    return Results.Ok(products);
})
.WithName("GetAllProducts")
.WithOpenApi();
```

Now, users can sort products by adding query parameters to the URL, such as:
- `/products?sortBy=name` - Sort products by name (ascending)
- `/products?sortBy=price&sortDesc=true` - Sort products by price (descending)
- `/products?sortBy=stock` - Sort products by stock quantity (ascending)

## Adding Pagination

To handle large result sets, let's add pagination to our endpoint:

```csharp
// Get all products with filtering, sorting, and pagination
app.MapGet("/products", async (
    JewelryContext db,
    int? categoryId,
    int? metalId,
    decimal? minPrice,
    decimal? maxPrice,
    bool? discounted,
    string? sortBy,
    bool? sortDesc,
    int page = 1,
    int pageSize = 10) =>
{
    var query = db.Products
        .Include(p => p.Metal)
        .Include(p => p.Category)
        .Include(p => p.Discount)
        .AsQueryable();

    // Apply filters
    if (categoryId.HasValue)
    {
        query = query.Where(p => p.CategoryId == categoryId.Value);
    }

    if (metalId.HasValue)
    {
        query = query.Where(p => p.MetalId == metalId.Value);
    }

    if (minPrice.HasValue)
    {
        query = query.Where(p => p.Price >= minPrice.Value);
    }

    if (maxPrice.HasValue)
    {
        query = query.Where(p => p.Price <= maxPrice.Value);
    }

    if (discounted.HasValue && discounted.Value)
    {
        query = query.Where(p => p.DiscountId != null && p.Discount.IsActive);
    }

    // Get total count for pagination
    var totalCount = await query.CountAsync();

    // Apply sorting
    if (!string.IsNullOrEmpty(sortBy))
    {
        bool isDescending = sortDesc ?? false;

        query = sortBy.ToLower() switch
        {
            "name" => isDescending
                ? query.OrderByDescending(p => p.Name)
                : query.OrderBy(p => p.Name),
            "price" => isDescending
                ? query.OrderByDescending(p => p.Price)
                : query.OrderBy(p => p.Price),
            "stock" => isDescending
                ? query.OrderByDescending(p => p.StockQuantity)
                : query.OrderBy(p => p.StockQuantity),
            _ => query.OrderBy(p => p.Id) // Default sorting
        };
    }
    else
    {
        // Default sorting by ID
        query = query.OrderBy(p => p.Id);
    }

    // Apply pagination
    var totalPages = (int)Math.Ceiling(totalCount / (double)pageSize);
    var products = await query
        .Skip((page - 1) * pageSize)
        .Take(pageSize)
        .ToListAsync();

    // Create pagination metadata
    var pagination = new
    {
        TotalCount = totalCount,
        TotalPages = totalPages,
        CurrentPage = page,
        PageSize = pageSize,
        HasPrevious = page > 1,
        HasNext = page < totalPages
    };

    return Results.Ok(new
    {
        Products = products,
        Pagination = pagination
    });
})
.WithName("GetAllProducts")
.WithOpenApi();
```

Now, users can paginate through the results by adding query parameters to the URL, such as:
- `/products?page=2` - Get the second page of products
- `/products?page=3&pageSize=20` - Get the third page with 20 products per page

The response now includes pagination metadata, which helps clients navigate through the result set.

## Formatting the Response

The current response includes all properties of the product, including navigation properties. This might result in a large response with more data than needed. Let's format the response to include only the necessary information:

```csharp
// Get all products with filtering, sorting, pagination, and response formatting
app.MapGet("/products", async (
    JewelryContext db,
    int? categoryId,
    int? metalId,
    decimal? minPrice,
    decimal? maxPrice,
    bool? discounted,
    string? sortBy,
    bool? sortDesc,
    int page = 1,
    int pageSize = 10) =>
{
    var query = db.Products
        .Include(p => p.Metal)
        .Include(p => p.Category)
        .Include(p => p.Discount)
        .AsQueryable();

    // Apply filters
    if (categoryId.HasValue)
    {
        query = query.Where(p => p.CategoryId == categoryId.Value);
    }

    if (metalId.HasValue)
    {
        query = query.Where(p => p.MetalId == metalId.Value);
    }

    if (minPrice.HasValue)
    {
        query = query.Where(p => p.Price >= minPrice.Value);
    }

    if (maxPrice.HasValue)
    {
        query = query.Where(p => p.Price <= maxPrice.Value);
    }

    if (discounted.HasValue && discounted.Value)
    {
        query = query.Where(p => p.DiscountId != null && p.Discount.IsActive);
    }

    // Get total count for pagination
    var totalCount = await query.CountAsync();

    // Apply sorting
    if (!string.IsNullOrEmpty(sortBy))
    {
        bool isDescending = sortDesc ?? false;

        query = sortBy.ToLower() switch
        {
            "name" => isDescending
                ? query.OrderByDescending(p => p.Name)
                : query.OrderBy(p => p.Name),
            "price" => isDescending
                ? query.OrderByDescending(p => p.Price)
                : query.OrderBy(p => p.Price),
            "stock" => isDescending
                ? query.OrderByDescending(p => p.StockQuantity)
                : query.OrderBy(p => p.StockQuantity),
            _ => query.OrderBy(p => p.Id) // Default sorting
        };
    }
    else
    {
        // Default sorting by ID
        query = query.OrderBy(p => p.Id);
    }

    // Apply pagination
    var totalPages = (int)Math.Ceiling(totalCount / (double)pageSize);
    var products = await query
        .Skip((page - 1) * pageSize)
        .Take(pageSize)
        .ToListAsync();

    // Format the response
    var formattedProducts = products.Select(p => new
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
            p.Metal.Id,
            p.Metal.Name
        },
        Category = new
        {
            p.Category.Id,
            p.Category.Name
        },
        Discount = p.Discount != null ? new
        {
            p.Discount.Id,
            p.Discount.Name,
            p.Discount.DiscountPercent,
            p.Discount.IsActive
        } : null
    });

    // Create pagination metadata
    var pagination = new
    {
        TotalCount = totalCount,
        TotalPages = totalPages,
        CurrentPage = page,
        PageSize = pageSize,
        HasPrevious = page > 1,
        HasNext = page < totalPages
    };

    return Results.Ok(new
    {
        Products = formattedProducts,
        Pagination = pagination
    });
})
.WithName("GetAllProducts")
.WithOpenApi();
```

Now, the response includes only the necessary information about each product, with nested objects for related entities like metal, category, and discount.

## Optimizing Database Queries

To optimize database queries, we can use projection to select only the properties we need, rather than loading entire entities:

```csharp
// Get all products with filtering, sorting, pagination, response formatting, and query optimization
app.MapGet("/products", async (
    JewelryContext db,
    int? categoryId,
    int? metalId,
    decimal? minPrice,
    decimal? maxPrice,
    bool? discounted,
    string? sortBy,
    bool? sortDesc,
    int page = 1,
    int pageSize = 10) =>
{
    var query = db.Products
        .AsNoTracking() // Improves performance for read-only queries
        .AsQueryable();

    // Apply filters
    if (categoryId.HasValue)
    {
        query = query.Where(p => p.CategoryId == categoryId.Value);
    }

    if (metalId.HasValue)
    {
        query = query.Where(p => p.MetalId == metalId.Value);
    }

    if (minPrice.HasValue)
    {
        query = query.Where(p => p.Price >= minPrice.Value);
    }

    if (maxPrice.HasValue)
    {
        query = query.Where(p => p.Price <= maxPrice.Value);
    }

    if (discounted.HasValue && discounted.Value)
    {
        query = query.Where(p => p.DiscountId != null && p.Discount.IsActive);
    }

    // Get total count for pagination
    var totalCount = await query.CountAsync();

    // Apply sorting
    if (!string.IsNullOrEmpty(sortBy))
    {
        bool isDescending = sortDesc ?? false;

        query = sortBy.ToLower() switch
        {
            "name" => isDescending
                ? query.OrderByDescending(p => p.Name)
                : query.OrderBy(p => p.Name),
            "price" => isDescending
                ? query.OrderByDescending(p => p.Price)
                : query.OrderBy(p => p.Price),
            "stock" => isDescending
                ? query.OrderByDescending(p => p.StockQuantity)
                : query.OrderBy(p => p.StockQuantity),
            _ => query.OrderBy(p => p.Id) // Default sorting
        };
    }
    else
    {
        // Default sorting by ID
        query = query.OrderBy(p => p.Id);
    }

    // Apply pagination and projection
    var totalPages = (int)Math.Ceiling(totalCount / (double)pageSize);
    var products = await query
        .Skip((page - 1) * pageSize)
        .Take(pageSize)
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
            Discount = p.Discount != null ? new
            {
                Id = p.Discount.Id,
                Name = p.Discount.Name,
                DiscountPercent = p.Discount.DiscountPercent,
                IsActive = p.Discount.IsActive
            } : null
        })
        .ToListAsync();

    // Create pagination metadata
    var pagination = new
    {
        TotalCount = totalCount,
        TotalPages = totalPages,
        CurrentPage = page,
        PageSize = pageSize,
        HasPrevious = page > 1,
        HasNext = page < totalPages
    };

    return Results.Ok(new
    {
        Products = products,
        Pagination = pagination
    });
})
.WithName("GetAllProducts")
.WithOpenApi();
```

In this optimized version:
- We use `AsNoTracking()` to improve performance for read-only queries
- We use projection with `Select()` to retrieve only the properties we need, rather than loading entire entities
- We perform the projection in the database query, rather than in memory

## Adding Search Functionality

Finally, let's add search functionality to our endpoint:

```csharp
// Get all products with filtering, sorting, pagination, response formatting, query optimization, and search
app.MapGet("/products", async (
    JewelryContext db,
    string? search,
    int? categoryId,
    int? metalId,
    decimal? minPrice,
    decimal? maxPrice,
    bool? discounted,
    string? sortBy,
    bool? sortDesc,
    int page = 1,
    int pageSize = 10) =>
{
    var query = db.Products
        .AsNoTracking()
        .AsQueryable();

    // Apply search
    if (!string.IsNullOrEmpty(search))
    {
        search = search.ToLower();
        query = query.Where(p =>
            p.Name.ToLower().Contains(search) ||
            p.Description.ToLower().Contains(search) ||
            p.Type.ToLower().Contains(search));
    }

    // Apply filters
    if (categoryId.HasValue)
    {
        query = query.Where(p => p.CategoryId == categoryId.Value);
    }

    if (metalId.HasValue)
    {
        query = query.Where(p => p.MetalId == metalId.Value);
    }

    if (minPrice.HasValue)
    {
        query = query.Where(p => p.Price >= minPrice.Value);
    }

    if (maxPrice.HasValue)
    {
        query = query.Where(p => p.Price <= maxPrice.Value);
    }

    if (discounted.HasValue && discounted.Value)
    {
        query = query.Where(p => p.DiscountId != null && p.Discount.IsActive);
    }

    // Get total count for pagination
    var totalCount = await query.CountAsync();

    // Apply sorting
    if (!string.IsNullOrEmpty(sortBy))
    {
        bool isDescending = sortDesc ?? false;

        query = sortBy.ToLower() switch
        {
            "name" => isDescending
                ? query.OrderByDescending(p => p.Name)
                : query.OrderBy(p => p.Name),
            "price" => isDescending
                ? query.OrderByDescending(p => p.Price)
                : query.OrderBy(p => p.Price),
            "stock" => isDescending
                ? query.OrderByDescending(p => p.StockQuantity)
                : query.OrderBy(p => p.StockQuantity),
            _ => query.OrderBy(p => p.Id) // Default sorting
        };
    }
    else
    {
        // Default sorting by ID
        query = query.OrderBy(p => p.Id);
    }

    // Apply pagination and projection
    var totalPages = (int)Math.Ceiling(totalCount / (double)pageSize);
    var products = await query
        .Skip((page - 1) * pageSize)
        .Take(pageSize)
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
            Discount = p.Discount != null ? new
            {
                Id = p.Discount.Id,
                Name = p.Discount.Name,
                DiscountPercent = p.Discount.DiscountPercent,
                IsActive = p.Discount.IsActive
            } : null
        })
        .ToListAsync();

    // Create pagination metadata
    var pagination = new
    {
        TotalCount = totalCount,
        TotalPages = totalPages,
        CurrentPage = page,
        PageSize = pageSize,
        HasPrevious = page > 1,
        HasNext = page < totalPages
    };

    return Results.Ok(new
    {
        Products = products,
        Pagination = pagination
    });
})
.WithName("GetAllProducts")
.WithOpenApi();
```

Now, users can search for products by adding a `search` query parameter to the URL, such as:
- `/products?search=diamond` - Search for products with "diamond" in the name, description, or type

## Conclusion

In this chapter, you've learned how to implement and enhance the endpoint for retrieving all products in the Jewelry Junction API. You've added filtering, sorting, pagination, search functionality, and response formatting. You've also learned how to optimize database queries for better performance.

In the next chapter, we'll implement the endpoint for retrieving a single product by ID, including its related data.

## Practice Exercise

Enhance your Get All Products endpoint by:
1. Adding a filter for products with low stock (e.g., `?lowStock=true` for products with stock quantity less than 5)
2. Adding a filter for products by gemstone (e.g., `?gemstoneId=1` for products with a specific gemstone)
3. Adding a sort option for average rating (e.g., `?sortBy=rating`)
4. Adding a filter for products with a minimum average rating (e.g., `?minRating=4`)
5. Implementing a more advanced search that also searches in metal and category names