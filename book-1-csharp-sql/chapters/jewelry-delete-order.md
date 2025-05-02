# Delete an Order

In this chapter, we'll implement the endpoint for deleting an order in our Jewelry Junction API. This will involve handling transactions, restoring product stock quantities, and implementing proper error handling.

## Learning Objectives

By the end of this chapter, you should be able to:
- Implement a DELETE endpoint to remove an order
- Use transactions to ensure data consistency
- Restore product stock quantities when an order is deleted
- Handle errors and edge cases
- Implement proper validation before deletion

## Understanding the Order Deletion Process

Deleting an order is not as simple as removing a record from the database. It involves several steps:
1. Validating that the order exists and can be deleted
2. Restoring product stock quantities for all items in the order
3. Removing order items associated with the order
4. Removing the order itself

Since these operations need to be atomic (all succeed or all fail), we'll use a transaction to ensure data consistency.

## Implementing the Endpoint

Let's implement the endpoint for deleting an order:

```csharp
// Delete an order
app.MapDelete("/orders/{id}", async (int id, JewelryContext db) =>
{
    using var transaction = await db.Database.BeginTransactionAsync();

    try
    {
        // Find the order with its items
        var order = await db.Orders
            .Include(o => o.OrderItems)
            .FirstOrDefaultAsync(o => o.Id == id);

        if (order == null)
        {
            return Results.NotFound(new { Message = $"Order with ID {id} not found" });
        }

        // Check if the order can be deleted (e.g., only pending orders can be deleted)
        if (order.Status != "Pending")
        {
            return Results.BadRequest(new { Message = $"Cannot delete order with status '{order.Status}'. Only pending orders can be deleted." });
        }

        // Restore product stock quantities
        foreach (var item in order.OrderItems)
        {
            var product = await db.Products.FindAsync(item.ProductId);
            if (product != null)
            {
                product.StockQuantity += item.Quantity;
            }
        }

        // Remove order items
        db.OrderItems.RemoveRange(order.OrderItems);

        // Remove the order
        db.Orders.Remove(order);

        // Save changes
        await db.SaveChangesAsync();

        // Commit the transaction
        await transaction.CommitAsync();

        return Results.Ok(new { Message = $"Order with ID {id} has been deleted" });
    }
    catch (Exception ex)
    {
        // Rollback the transaction
        await transaction.RollbackAsync();

        // Log the exception
        Console.WriteLine($"Error deleting order: {ex.Message}");

        // Return a 500 Internal Server Error response
        return Results.Problem(
            title: "An error occurred while deleting the order",
            detail: ex.Message,
            statusCode: 500
        );
    }
})
.WithName("DeleteOrder")
.WithOpenApi();
```

This endpoint:
1. Begins a transaction to ensure data consistency
2. Finds the order with its items
3. Checks if the order exists and can be deleted
4. Restores product stock quantities for all items in the order
5. Removes order items associated with the order
6. Removes the order itself
7. Commits the transaction

If any step fails, the transaction is rolled back, ensuring that no partial changes are made to the database.

## Enhancing the Endpoint

Let's enhance the endpoint to provide more information about the deleted order and to handle more edge cases:

```csharp
// Delete an order with enhanced response
app.MapDelete("/orders/{id}", async (int id, JewelryContext db) =>
{
    using var transaction = await db.Database.BeginTransactionAsync();

    try
    {
        // Find the order with its items and customer
        var order = await db.Orders
            .Include(o => o.OrderItems)
                .ThenInclude(oi => oi.Product)
            .Include(o => o.Customer)
            .FirstOrDefaultAsync(o => o.Id == id);

        if (order == null)
        {
            return Results.NotFound(new { Message = $"Order with ID {id} not found" });
        }

        // Check if the order can be deleted (e.g., only pending orders can be deleted)
        if (order.Status != "Pending")
        {
            return Results.BadRequest(new { Message = $"Cannot delete order with status '{order.Status}'. Only pending orders can be deleted." });
        }

        // Prepare response data before deletion
        var orderDetails = new
        {
            order.Id,
            order.OrderDate,
            order.Status,
            order.TotalAmount,
            Customer = new
            {
                order.Customer.Id,
                Name = $"{order.Customer.FirstName} {order.Customer.LastName}",
                order.Customer.Email
            },
            Items = order.OrderItems.Select(oi => new
            {
                oi.Id,
                oi.ProductId,
                ProductName = oi.Product.Name,
                oi.Quantity,
                oi.UnitPrice,
                Subtotal = oi.Quantity * oi.UnitPrice
            }).ToList(),
            ItemCount = order.OrderItems.Count
        };

        // Restore product stock quantities
        foreach (var item in order.OrderItems)
        {
            var product = await db.Products.FindAsync(item.ProductId);
            if (product != null)
            {
                product.StockQuantity += item.Quantity;
            }
        }

        // Remove order items
        db.OrderItems.RemoveRange(order.OrderItems);

        // Remove the order
        db.Orders.Remove(order);

        // Save changes
        await db.SaveChangesAsync();

        // Commit the transaction
        await transaction.CommitAsync();

        return Results.Ok(new
        {
            Message = $"Order with ID {id} has been deleted",
            DeletedOrder = orderDetails
        });
    }
    catch (Exception ex)
    {
        // Rollback the transaction
        await transaction.RollbackAsync();

        // Log the exception
        Console.WriteLine($"Error deleting order: {ex.Message}");

        // Return a 500 Internal Server Error response
        return Results.Problem(
            title: "An error occurred while deleting the order",
            detail: ex.Message,
            statusCode: 500
        );
    }
})
.WithName("DeleteOrder")
.WithOpenApi();
```

This enhanced endpoint provides more information about the deleted order, including customer details and item details. It also includes the total amount and item count.

## Implementing Soft Delete

In many real-world applications, it's preferable to implement "soft delete" rather than actually removing records from the database. This involves marking records as deleted rather than physically removing them, which allows for data recovery and audit trails.

Let's modify our approach to implement soft delete for orders:

1. First, let's add a `IsDeleted` property to the `Order` class:

```csharp
public class Order
{
    public int Id { get; set; }
    public DateTime OrderDate { get; set; }
    public decimal TotalAmount { get; set; }
    public string Status { get; set; } = string.Empty;
    public bool IsDeleted { get; set; } = false;
    public DateTime? DeletedAt { get; set; }

    // Relationships
    public int CustomerId { get; set; }
    public Customer? Customer { get; set; }
    public List<OrderItem> OrderItems { get; set; } = new List<OrderItem>();
}
```

2. Now, let's update our endpoint to implement soft delete:

```csharp
// Soft delete an order
app.MapDelete("/orders/{id}", async (int id, JewelryContext db) =>
{
    using var transaction = await db.Database.BeginTransactionAsync();

    try
    {
        // Find the order with its items and customer
        var order = await db.Orders
            .Include(o => o.OrderItems)
                .ThenInclude(oi => oi.Product)
            .Include(o => o.Customer)
            .FirstOrDefaultAsync(o => o.Id == id);

        if (order == null)
        {
            return Results.NotFound(new { Message = $"Order with ID {id} not found" });
        }

        // Check if the order is already deleted
        if (order.IsDeleted)
        {
            return Results.BadRequest(new { Message = $"Order with ID {id} is already deleted" });
        }

        // Check if the order can be deleted (e.g., only pending orders can be deleted)
        if (order.Status != "Pending")
        {
            return Results.BadRequest(new { Message = $"Cannot delete order with status '{order.Status}'. Only pending orders can be deleted." });
        }

        // Prepare response data before deletion
        var orderDetails = new
        {
            order.Id,
            order.OrderDate,
            order.Status,
            order.TotalAmount,
            Customer = new
            {
                order.Customer.Id,
                Name = $"{order.Customer.FirstName} {order.Customer.LastName}",
                order.Customer.Email
            },
            Items = order.OrderItems.Select(oi => new
            {
                oi.Id,
                oi.ProductId,
                ProductName = oi.Product.Name,
                oi.Quantity,
                oi.UnitPrice,
                Subtotal = oi.Quantity * oi.UnitPrice
            }).ToList(),
            ItemCount = order.OrderItems.Count
        };

        // Restore product stock quantities
        foreach (var item in order.OrderItems)
        {
            var product = await db.Products.FindAsync(item.ProductId);
            if (product != null)
            {
                product.StockQuantity += item.Quantity;
            }
        }

        // Mark the order as deleted
        order.IsDeleted = true;
        order.DeletedAt = DateTime.Now;
        order.Status = "Cancelled";

        // Save changes
        await db.SaveChangesAsync();

        // Commit the transaction
        await transaction.CommitAsync();

        return Results.Ok(new
        {
            Message = $"Order with ID {id} has been deleted",
            DeletedOrder = orderDetails
        });
    }
    catch (Exception ex)
    {
        // Rollback the transaction
        await transaction.RollbackAsync();

        // Log the exception
        Console.WriteLine($"Error deleting order: {ex.Message}");

        // Return a 500 Internal Server Error response
        return Results.Problem(
            title: "An error occurred while deleting the order",
            detail: ex.Message,
            statusCode: 500
        );
    }
})
.WithName("DeleteOrder")
.WithOpenApi();
```

3. We also need to update our queries to filter out deleted orders:

```csharp
// Get all orders (excluding deleted orders)
app.MapGet("/orders", async (JewelryContext db) =>
    await db.Orders
        .Where(o => !o.IsDeleted)
        .Include(o => o.Customer)
        .Include(o => o.OrderItems)
        .ToListAsync())
    .WithName("GetAllOrders")
    .WithOpenApi();

// Get order by ID (excluding deleted orders)
app.MapGet("/orders/{id}", async (int id, JewelryContext db) =>
{
    var order = await db.Orders
        .Where(o => !o.IsDeleted)
        .Include(o => o.Customer)
        .Include(o => o.OrderItems)
            .ThenInclude(oi => oi.Product)
        .FirstOrDefaultAsync(o => o.Id == id);

    return order != null ? Results.Ok(order) : Results.NotFound();
})
.WithName("GetOrderById")
.WithOpenApi();
```

## Implementing Hard Delete with Confirmation

In some cases, you might want to allow hard deletion of orders, but with an additional confirmation step to prevent accidental deletions. Let's implement an endpoint for hard deletion with confirmation:

```csharp
// Hard delete an order with confirmation
app.MapDelete("/orders/{id}/hard", async (int id, string confirmation, JewelryContext db) =>
{
    // Check if confirmation is provided
    if (string.IsNullOrEmpty(confirmation) || confirmation != "CONFIRM_DELETE")
    {
        return Results.BadRequest(new { Message = "Confirmation is required. Please provide 'CONFIRM_DELETE' as the confirmation parameter." });
    }

    using var transaction = await db.Database.BeginTransactionAsync();

    try
    {
        // Find the order with its items
        var order = await db.Orders
            .Include(o => o.OrderItems)
            .FirstOrDefaultAsync(o => o.Id == id);

        if (order == null)
        {
            return Results.NotFound(new { Message = $"Order with ID {id} not found" });
        }

        // Restore product stock quantities
        foreach (var item in order.OrderItems)
        {
            var product = await db.Products.FindAsync(item.ProductId);
            if (product != null)
            {
                product.StockQuantity += item.Quantity;
            }
        }

        // Remove order items
        db.OrderItems.RemoveRange(order.OrderItems);

        // Remove the order
        db.Orders.Remove(order);

        // Save changes
        await db.SaveChangesAsync();

        // Commit the transaction
        await transaction.CommitAsync();

        return Results.Ok(new { Message = $"Order with ID {id} has been permanently deleted" });
    }
    catch (Exception ex)
    {
        // Rollback the transaction
        await transaction.RollbackAsync();

        // Log the exception
        Console.WriteLine($"Error deleting order: {ex.Message}");

        // Return a 500 Internal Server Error response
        return Results.Problem(
            title: "An error occurred while deleting the order",
            detail: ex.Message,
            statusCode: 500
        );
    }
})
.WithName("HardDeleteOrder")
.WithOpenApi();
```

This endpoint requires a confirmation parameter with the value "CONFIRM_DELETE" to proceed with the hard deletion. This helps prevent accidental deletions.

## Implementing a Restore Endpoint

If we're using soft delete, it's also useful to provide an endpoint to restore deleted orders:

```csharp
// Restore a deleted order
app.MapPost("/orders/{id}/restore", async (int id, JewelryContext db) =>
{
    using var transaction = await db.Database.BeginTransactionAsync();

    try
    {
        // Find the order
        var order = await db.Orders
            .Include(o => o.OrderItems)
            .FirstOrDefaultAsync(o => o.Id == id);

        if (order == null)
        {
            return Results.NotFound(new { Message = $"Order with ID {id} not found" });
        }

        // Check if the order is deleted
        if (!order.IsDeleted)
        {
            return Results.BadRequest(new { Message = $"Order with ID {id} is not deleted" });
        }

        // Deduct product stock quantities again
        foreach (var item in order.OrderItems)
        {
            var product = await db.Products.FindAsync(item.ProductId);
            if (product != null)
            {
                // Check if there's enough stock
                if (product.StockQuantity < item.Quantity)
                {
                    return Results.BadRequest(new { Message = $"Insufficient stock for product {product.Name}. Available: {product.StockQuantity}, Required: {item.Quantity}" });
                }

                product.StockQuantity -= item.Quantity;
            }
        }

        // Restore the order
        order.IsDeleted = false;
        order.DeletedAt = null;
        order.Status = "Pending";

        // Save changes
        await db.SaveChangesAsync();

        // Commit the transaction
        await transaction.CommitAsync();

        return Results.Ok(new { Message = $"Order with ID {id} has been restored" });
    }
    catch (Exception ex)
    {
        // Rollback the transaction
        await transaction.RollbackAsync();

        // Log the exception
        Console.WriteLine($"Error restoring order: {ex.Message}");

        // Return a 500 Internal Server Error response
        return Results.Problem(
            title: "An error occurred while restoring the order",
            detail: ex.Message,
            statusCode: 500
        );
    }
})
.WithName("RestoreOrder")
.WithOpenApi();
```

This endpoint allows restoring a deleted order, but only if there's enough stock available for all items in the order.

## Conclusion

In this chapter, you've learned how to implement endpoints for deleting orders in the Jewelry Junction API. You've explored different approaches to deletion, including hard delete and soft delete, and you've learned how to handle transactions, restore product stock quantities, and implement proper error handling. You've also learned how to implement a restore endpoint for soft-deleted orders.

In the next chapter, we'll implement the endpoint for updating a product, which will involve validation and handling of related data.

## Practice Exercise

Enhance your order deletion functionality by:
1. Adding an audit trail for order deletions (who deleted it, when, why)
2. Implementing a "recycle bin" feature that allows viewing and restoring deleted orders
3. Adding a scheduled task to permanently delete orders that have been soft-deleted for more than 30 days
4. Adding support for partial order cancellation (cancelling specific items in an order)
5. Implementing different deletion rules based on the order status (e.g., pending orders can be deleted, shipped orders can only be marked as returned)