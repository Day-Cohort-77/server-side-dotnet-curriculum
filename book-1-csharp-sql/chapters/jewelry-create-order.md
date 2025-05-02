# Create New Order

In this chapter, we'll implement the endpoint for creating a new order in our Jewelry Junction API. This will involve more complex operations like transactions, validation, and updating related data.

## Learning Objectives

By the end of this chapter, you should be able to:
- Implement a POST endpoint to create a new order
- Validate the order data before processing
- Use transactions to ensure data consistency
- Update related data (product stock quantities)
- Handle errors and edge cases
- Return appropriate responses

## Understanding the Order Creation Process

Creating an order is a complex operation that involves several steps:
1. Validating the order data (customer exists, products exist, sufficient stock, etc.)
2. Creating the order record
3. Creating order item records for each product in the order
4. Updating product stock quantities
5. Calculating the total order amount
6. Returning the created order with its items

Since these operations need to be atomic (all succeed or all fail), we'll use a transaction to ensure data consistency.

## Implementing the Endpoint

Let's implement the endpoint for creating a new order:

```csharp
// Create a new order
app.MapPost("/orders", async (Order order, JewelryContext db) =>
{
    using var transaction = await db.Database.BeginTransactionAsync();

    try
    {
        // Validate customer exists
        var customer = await db.Customers.FindAsync(order.CustomerId);
        if (customer == null)
        {
            return Results.BadRequest(new { Message = $"Customer with ID {order.CustomerId} not found" });
        }

        // Set order date to current date if not provided
        if (order.OrderDate == default)
        {
            order.OrderDate = DateTime.Now;
        }

        // Set initial status if not provided
        if (string.IsNullOrEmpty(order.Status))
        {
            order.Status = "Pending";
        }

        // Validate order items
        if (order.OrderItems == null || !order.OrderItems.Any())
        {
            return Results.BadRequest(new { Message = "Order must contain at least one item" });
        }

        // Validate products and check stock
        decimal totalAmount = 0;
        foreach (var item in order.OrderItems)
        {
            var product = await db.Products
                .Include(p => p.Discount)
                .FirstOrDefaultAsync(p => p.Id == item.ProductId);

            if (product == null)
            {
                return Results.BadRequest(new { Message = $"Product with ID {item.ProductId} not found" });
            }

            if (product.StockQuantity < item.Quantity)
            {
                return Results.BadRequest(new { Message = $"Insufficient stock for product {product.Name}. Available: {product.StockQuantity}, Requested: {item.Quantity}" });
            }

            // Calculate item price (considering discounts)
            decimal unitPrice = product.Price;
            if (product.Discount != null && product.Discount.IsActive)
            {
                unitPrice = product.Price * (1 - product.Discount.DiscountPercent / 100);
            }

            // Set the unit price in the order item
            item.UnitPrice = unitPrice;

            // Add to total amount
            totalAmount += unitPrice * item.Quantity;

            // Update product stock quantity
            product.StockQuantity -= item.Quantity;
        }

        // Set the total amount
        order.TotalAmount = totalAmount;

        // Add the order to the database
        db.Orders.Add(order);
        await db.SaveChangesAsync();

        // Commit the transaction
        await transaction.CommitAsync();

        // Return the created order
        return Results.Created($"/orders/{order.Id}", order);
    }
    catch (Exception ex)
    {
        // Rollback the transaction
        await transaction.RollbackAsync();

        // Log the exception
        Console.WriteLine($"Error creating order: {ex.Message}");

        // Return a 500 Internal Server Error response
        return Results.Problem(
            title: "An error occurred while creating the order",
            detail: ex.Message,
            statusCode: 500
        );
    }
})
.WithName("CreateOrder")
.WithOpenApi();
```

This endpoint:
1. Begins a transaction to ensure data consistency
2. Validates that the customer exists
3. Sets default values for order date and status if not provided
4. Validates that the order contains at least one item
5. Validates that all products exist and have sufficient stock
6. Calculates the unit price for each item, considering discounts
7. Updates product stock quantities
8. Calculates the total order amount
9. Adds the order to the database
10. Commits the transaction
11. Returns the created order

If any step fails, the transaction is rolled back, ensuring that no partial changes are made to the database.

## Enhancing the Endpoint

Let's enhance the endpoint to provide a more structured response and to include more information about the created order:

```csharp
// Create a new order with enhanced response
app.MapPost("/orders", async (Order order, JewelryContext db) =>
{
    using var transaction = await db.Database.BeginTransactionAsync();

    try
    {
        // Validate customer exists
        var customer = await db.Customers.FindAsync(order.CustomerId);
        if (customer == null)
        {
            return Results.BadRequest(new { Message = $"Customer with ID {order.CustomerId} not found" });
        }

        // Set order date to current date if not provided
        if (order.OrderDate == default)
        {
            order.OrderDate = DateTime.Now;
        }

        // Set initial status if not provided
        if (string.IsNullOrEmpty(order.Status))
        {
            order.Status = "Pending";
        }

        // Validate order items
        if (order.OrderItems == null || !order.OrderItems.Any())
        {
            return Results.BadRequest(new { Message = "Order must contain at least one item" });
        }

        // Validate products and check stock
        decimal totalAmount = 0;
        var orderItemDetails = new List<object>();

        foreach (var item in order.OrderItems)
        {
            var product = await db.Products
                .Include(p => p.Discount)
                .Include(p => p.Metal)
                .Include(p => p.Category)
                .FirstOrDefaultAsync(p => p.Id == item.ProductId);

            if (product == null)
            {
                return Results.BadRequest(new { Message = $"Product with ID {item.ProductId} not found" });
            }

            if (product.StockQuantity < item.Quantity)
            {
                return Results.BadRequest(new { Message = $"Insufficient stock for product {product.Name}. Available: {product.StockQuantity}, Requested: {item.Quantity}" });
            }

            // Calculate item price (considering discounts)
            decimal unitPrice = product.Price;
            decimal? discountAmount = null;

            if (product.Discount != null && product.Discount.IsActive)
            {
                discountAmount = product.Price * (product.Discount.DiscountPercent / 100);
                unitPrice = product.Price - discountAmount.Value;
            }

            // Set the unit price in the order item
            item.UnitPrice = unitPrice;

            // Add to total amount
            totalAmount += unitPrice * item.Quantity;

            // Update product stock quantity
            product.StockQuantity -= item.Quantity;

            // Add item details for the response
            orderItemDetails.Add(new
            {
                item.Id,
                item.ProductId,
                ProductName = product.Name,
                ProductType = product.Type,
                Metal = product.Metal.Name,
                Category = product.Category.Name,
                OriginalPrice = product.Price,
                DiscountAmount = discountAmount,
                item.UnitPrice,
                item.Quantity,
                Subtotal = unitPrice * item.Quantity
            });
        }

        // Set the total amount
        order.TotalAmount = totalAmount;

        // Add the order to the database
        db.Orders.Add(order);
        await db.SaveChangesAsync();

        // Commit the transaction
        await transaction.CommitAsync();

        // Prepare the response
        var response = new
        {
            order.Id,
            order.OrderDate,
            order.Status,
            order.TotalAmount,
            Customer = new
            {
                customer.Id,
                Name = $"{customer.FirstName} {customer.LastName}",
                customer.Email,
                customer.Phone,
                customer.Address
            },
            Items = orderItemDetails,
            ItemCount = order.OrderItems.Count,
            CreatedAt = DateTime.Now
        };

        // Return the created order
        return Results.Created($"/orders/{order.Id}", response);
    }
    catch (Exception ex)
    {
        // Rollback the transaction
        await transaction.RollbackAsync();

        // Log the exception
        Console.WriteLine($"Error creating order: {ex.Message}");

        // Return a 500 Internal Server Error response
        return Results.Problem(
            title: "An error occurred while creating the order",
            detail: ex.Message,
            statusCode: 500
        );
    }
})
.WithName("CreateOrder")
.WithOpenApi();
```

This enhanced endpoint provides a more structured response that includes:
- Basic order information (ID, date, status, total amount)
- Customer information
- Detailed information about each order item, including product details and pricing
- Summary information (item count, creation timestamp)

## Implementing Order Validation

To make our code more maintainable, let's extract the order validation logic into a separate method:

```csharp
// Validate order
private static async Task<(bool IsValid, string? ErrorMessage, Customer? Customer)> ValidateOrderAsync(Order order, JewelryContext db)
{
    // Validate customer exists
    var customer = await db.Customers.FindAsync(order.CustomerId);
    if (customer == null)
    {
        return (false, $"Customer with ID {order.CustomerId} not found", null);
    }

    // Validate order items
    if (order.OrderItems == null || !order.OrderItems.Any())
    {
        return (false, "Order must contain at least one item", null);
    }

    return (true, null, customer);
}

// Validate order items
private static async Task<(bool IsValid, string? ErrorMessage, List<(OrderItem Item, Product Product, decimal UnitPrice, decimal? DiscountAmount)> ValidItems)> ValidateOrderItemsAsync(List<OrderItem> orderItems, JewelryContext db)
{
    var validItems = new List<(OrderItem Item, Product Product, decimal UnitPrice, decimal? DiscountAmount)>();

    foreach (var item in orderItems)
    {
        var product = await db.Products
            .Include(p => p.Discount)
            .Include(p => p.Metal)
            .Include(p => p.Category)
            .FirstOrDefaultAsync(p => p.Id == item.ProductId);

        if (product == null)
        {
            return (false, $"Product with ID {item.ProductId} not found", validItems);
        }

        if (product.StockQuantity < item.Quantity)
        {
            return (false, $"Insufficient stock for product {product.Name}. Available: {product.StockQuantity}, Requested: {item.Quantity}", validItems);
        }

        // Calculate item price (considering discounts)
        decimal unitPrice = product.Price;
        decimal? discountAmount = null;

        if (product.Discount != null && product.Discount.IsActive)
        {
            discountAmount = product.Price * (product.Discount.DiscountPercent / 100);
            unitPrice = product.Price - discountAmount.Value;
        }

        validItems.Add((item, product, unitPrice, discountAmount));
    }

    return (true, null, validItems);
}
```

Now, let's update our endpoint to use these validation methods:

```csharp
// Create a new order with validation methods
app.MapPost("/orders", async (Order order, JewelryContext db) =>
{
    using var transaction = await db.Database.BeginTransactionAsync();

    try
    {
        // Set order date to current date if not provided
        if (order.OrderDate == default)
        {
            order.OrderDate = DateTime.Now;
        }

        // Set initial status if not provided
        if (string.IsNullOrEmpty(order.Status))
        {
            order.Status = "Pending";
        }

        // Validate order
        var (isOrderValid, orderErrorMessage, customer) = await ValidateOrderAsync(order, db);
        if (!isOrderValid)
        {
            return Results.BadRequest(new { Message = orderErrorMessage });
        }

        // Validate order items
        var (areItemsValid, itemsErrorMessage, validItems) = await ValidateOrderItemsAsync(order.OrderItems, db);
        if (!areItemsValid)
        {
            return Results.BadRequest(new { Message = itemsErrorMessage });
        }

        // Process order items
        decimal totalAmount = 0;
        var orderItemDetails = new List<object>();

        foreach (var (item, product, unitPrice, discountAmount) in validItems)
        {
            // Set the unit price in the order item
            item.UnitPrice = unitPrice;

            // Add to total amount
            totalAmount += unitPrice * item.Quantity;

            // Update product stock quantity
            product.StockQuantity -= item.Quantity;

            // Add item details for the response
            orderItemDetails.Add(new
            {
                item.Id,
                item.ProductId,
                ProductName = product.Name,
                ProductType = product.Type,
                Metal = product.Metal.Name,
                Category = product.Category.Name,
                OriginalPrice = product.Price,
                DiscountAmount = discountAmount,
                item.UnitPrice,
                item.Quantity,
                Subtotal = unitPrice * item.Quantity
            });
        }

        // Set the total amount
        order.TotalAmount = totalAmount;

        // Add the order to the database
        db.Orders.Add(order);
        await db.SaveChangesAsync();

        // Commit the transaction
        await transaction.CommitAsync();

        // Prepare the response
        var response = new
        {
            order.Id,
            order.OrderDate,
            order.Status,
            order.TotalAmount,
            Customer = new
            {
                customer.Id,
                Name = $"{customer.FirstName} {customer.LastName}",
                customer.Email,
                customer.Phone,
                customer.Address
            },
            Items = orderItemDetails,
            ItemCount = order.OrderItems.Count,
            CreatedAt = DateTime.Now
        };

        // Return the created order
        return Results.Created($"/orders/{order.Id}", response);
    }
    catch (Exception ex)
    {
        // Rollback the transaction
        await transaction.RollbackAsync();

        // Log the exception
        Console.WriteLine($"Error creating order: {ex.Message}");

        // Return a 500 Internal Server Error response
        return Results.Problem(
            title: "An error occurred while creating the order",
            detail: ex.Message,
            statusCode: 500
        );
    }
})
.WithName("CreateOrder")
.WithOpenApi();
```

This approach makes the code more maintainable and easier to understand by separating the validation logic from the main endpoint logic.

## Implementing a Request Model

To further improve our API, let's create a request model for creating orders. This will allow us to separate the API contract from our internal data model:

```csharp
// Order creation request model
public class CreateOrderRequest
{
    public int CustomerId { get; set; }
    public List<CreateOrderItemRequest> Items { get; set; } = new List<CreateOrderItemRequest>();
}

public class CreateOrderItemRequest
{
    public int ProductId { get; set; }
    public int Quantity { get; set; }
}
```

Now, let's update our endpoint to use this request model:

```csharp
// Create a new order with request model
app.MapPost("/orders", async (CreateOrderRequest request, JewelryContext db) =>
{
    using var transaction = await db.Database.BeginTransactionAsync();

    try
    {
        // Validate customer exists
        var customer = await db.Customers.FindAsync(request.CustomerId);
        if (customer == null)
        {
            return Results.BadRequest(new { Message = $"Customer with ID {request.CustomerId} not found" });
        }

        // Validate order items
        if (request.Items == null || !request.Items.Any())
        {
            return Results.BadRequest(new { Message = "Order must contain at least one item" });
        }

        // Create order
        var order = new Order
        {
            CustomerId = request.CustomerId,
            OrderDate = DateTime.Now,
            Status = "Pending",
            OrderItems = new List<OrderItem>()
        };

        // Process order items
        decimal totalAmount = 0;
        var orderItemDetails = new List<object>();

        foreach (var itemRequest in request.Items)
        {
            var product = await db.Products
                .Include(p => p.Discount)
                .Include(p => p.Metal)
                .Include(p => p.Category)
                .FirstOrDefaultAsync(p => p.Id == itemRequest.ProductId);

            if (product == null)
            {
                return Results.BadRequest(new { Message = $"Product with ID {itemRequest.ProductId} not found" });
            }

            if (product.StockQuantity < itemRequest.Quantity)
            {
                return Results.BadRequest(new { Message = $"Insufficient stock for product {product.Name}. Available: {product.StockQuantity}, Requested: {itemRequest.Quantity}" });
            }

            // Calculate item price (considering discounts)
            decimal unitPrice = product.Price;
            decimal? discountAmount = null;

            if (product.Discount != null && product.Discount.IsActive)
            {
                discountAmount = product.Price * (product.Discount.DiscountPercent / 100);
                unitPrice = product.Price - discountAmount.Value;
            }

            // Create order item
            var orderItem = new OrderItem
            {
                ProductId = itemRequest.ProductId,
                Quantity = itemRequest.Quantity,
                UnitPrice = unitPrice
            };

            // Add to order
            order.OrderItems.Add(orderItem);

            // Add to total amount
            totalAmount += unitPrice * itemRequest.Quantity;

            // Update product stock quantity
            product.StockQuantity -= itemRequest.Quantity;

            // Add item details for the response
            orderItemDetails.Add(new
            {
                ProductId = product.Id,
                ProductName = product.Name,
                ProductType = product.Type,
                Metal = product.Metal.Name,
                Category = product.Category.Name,
                OriginalPrice = product.Price,
                DiscountAmount = discountAmount,
                UnitPrice = unitPrice,
                Quantity = itemRequest.Quantity,
                Subtotal = unitPrice * itemRequest.Quantity
            });
        }

        // Set the total amount
        order.TotalAmount = totalAmount;

        // Add the order to the database
        db.Orders.Add(order);
        await db.SaveChangesAsync();

        // Commit the transaction
        await transaction.CommitAsync();

        // Prepare the response
        var response = new
        {
            order.Id,
            order.OrderDate,
            order.Status,
            order.TotalAmount,
            Customer = new
            {
                customer.Id,
                Name = $"{customer.FirstName} {customer.LastName}",
                customer.Email,
                customer.Phone,
                customer.Address
            },
            Items = orderItemDetails,
            ItemCount = order.OrderItems.Count,
            CreatedAt = DateTime.Now
        };

        // Return the created order
        return Results.Created($"/orders/{order.Id}", response);
    }
    catch (Exception ex)
    {
        // Rollback the transaction
        await transaction.RollbackAsync();

        // Log the exception
        Console.WriteLine($"Error creating order: {ex.Message}");

        // Return a 500 Internal Server Error response
        return Results.Problem(
            title: "An error occurred while creating the order",
            detail: ex.Message,
            statusCode: 500
        );
    }
})
.WithName("CreateOrder")
.WithOpenApi();
```

Using a request model provides several benefits:
- It separates the API contract from the internal data model
- It makes the API more flexible and easier to evolve
- It provides a clear interface for clients to understand what data is required
- It allows for validation specific to the API contract

## Conclusion

In this chapter, you've learned how to implement the endpoint for creating a new order in the Jewelry Junction API. You've used transactions to ensure data consistency, validated the order data before processing, updated related data, and handled errors and edge cases. You've also learned how to structure the response to provide useful information to clients and how to use request models to separate the API contract from the internal data model.

In the next chapter, we'll implement the endpoint for deleting an order, which will involve similar concepts like transactions and validation.

## Practice Exercise

Enhance your Create Order endpoint by:
1. Adding validation for the quantity (must be positive)
2. Adding support for applying a discount code to the entire order
3. Adding support for shipping information (address, shipping method, etc.)
4. Implementing inventory reservation (temporarily reserve inventory during checkout)
5. Adding support for payment information (payment method, transaction ID, etc.)