# Order Management Endpoints

In this chapter, we'll implement order management endpoints for our TinyTreats application. These endpoints will allow customers to place orders, view their order history, and allow staff to manage orders.

## Understanding Order Management

Order management is a critical feature of our bakery management system. It involves:

1. **Creating orders**: Customers placing new orders
2. **Reading orders**: Viewing order details and history
3. **Updating orders**: Staff updating order status
4. **Managing order items**: Adding, removing, or modifying items in an order

We'll implement these operations as RESTful endpoints, following our organized approach of placing related endpoints in separate files.

## Creating the Order Endpoints

Let's create a new file called `Endpoints/OrderEndpoints.cs` to contain all our order-related endpoints:

```csharp
// Endpoints/OrderEndpoints.cs
using Microsoft.AspNetCore.Identity;
using Microsoft.EntityFrameworkCore;
using System.Security.Claims;
using TinyTreats.Data;
using TinyTreats.DTO;
using AutoMapper;
using AutoMapper.QueryableExtensions;
using TinyTreats.Models;

namespace TinyTreats.Endpoints;

public static class OrderEndpoints
{
    public static void MapOrderEndpoints(this WebApplication app)
    {
        // Get all orders - Admin can see all, Baker can see all, Customer can see only their own
        app.MapGet("/orders", async (
            ClaimsPrincipal user,
            UserManager<IdentityUser> userManager,
            TinyTreatsDbContext dbContext,
            IMapper mapper) =>
        {
            // Get the current user's ID
            var userId = user.FindFirstValue(ClaimTypes.NameIdentifier);
            if (userId == null)
            {
                return Results.Unauthorized();
            }

            // Check if the user is an admin or baker
            bool isAdmin = user.IsInRole("Admin");
            bool isBaker = user.IsInRole("Baker");

            // Get the user's profile
            var userProfile = await dbContext.UserProfiles
                .FirstOrDefaultAsync(up => up.IdentityUserId == userId);

            if (userProfile == null)
            {
                return Results.NotFound("User profile not found");
            }

            // Query orders based on role
            IQueryable<Order> ordersQuery = dbContext.Orders
                .Include(o => o.OrderItems)
                .ThenInclude(oi => oi.Product)
                .Include(o => o.UserProfile);

            if (!isAdmin && !isBaker)
            {
                // Customers can only see their own orders
                ordersQuery = ordersQuery.Where(o => o.UserProfileId == userProfile.Id);
            }

            var orders = await ordersQuery.ToListAsync();

            // Map to DTOs using AutoMapper
            var orderDtos = mapper.Map<List<OrderDto>>(orders);

            return Results.Ok(orderDtos);
        }).RequireAuthorization();

        // Get order by ID - Admin can see any, Baker can see any, Customer can see only their own
        app.MapGet("/orders/{id}", async (
            int id,
            ClaimsPrincipal user,
            UserManager<IdentityUser> userManager,
            TinyTreatsDbContext dbContext,
            IMapper mapper) =>
        {
            // Get the current user's ID
            var userId = user.FindFirstValue(ClaimTypes.NameIdentifier);
            if (userId == null)
            {
                return Results.Unauthorized();
            }

            // Check if the user is an admin or baker
            bool isAdmin = user.IsInRole("Admin");
            bool isBaker = user.IsInRole("Baker");

            // Get the order with related data
            var order = await dbContext.Orders
                .Include(o => o.OrderItems)
                .ThenInclude(oi => oi.Product)
                .Include(o => o.UserProfile)
                .FirstOrDefaultAsync(o => o.Id == id);

            if (order == null)
            {
                return Results.NotFound();
            }

            // Check if the user has permission to view this order
            if (!isAdmin && !isBaker)
            {
                // Get the user's profile
                var userProfile = await dbContext.UserProfiles
                    .FirstOrDefaultAsync(up => up.IdentityUserId == userId);

                if (userProfile == null || order.UserProfileId != userProfile.Id)
                {
                    return Results.NotFound(); // Use NotFound instead of Forbidden for security
                }
            }

            // Map to DTO using AutoMapper
            var orderDto = mapper.Map<OrderDto>(order);

            return Results.Ok(orderDto);
        }).RequireAuthorization();

        // Create a new order - Any authenticated user
        app.MapPost("/orders", async (
            OrderCreateDto orderDto,
            ClaimsPrincipal user,
            UserManager<IdentityUser> userManager,
            TinyTreatsDbContext dbContext,
            IMapper mapper) =>
        {
            // Get the current user's ID
            var userId = user.FindFirstValue(ClaimTypes.NameIdentifier);
            if (userId == null)
            {
                return Results.Unauthorized();
            }

            // Get the user's profile
            var userProfile = await dbContext.UserProfiles
                .FirstOrDefaultAsync(up => up.IdentityUserId == userId);

            if (userProfile == null)
            {
                return Results.NotFound("User profile not found");
            }

            // Validate that all products exist and are available
            var productIds = orderDto.Items.Select(i => i.ProductId).ToList();
            var products = await dbContext.Products
                .Where(p => productIds.Contains(p.Id) && p.IsAvailable)
                .ToDictionaryAsync(p => p.Id, p => p);

            // Check if all products were found and are available
            if (products.Count != productIds.Count)
            {
                return Results.BadRequest("One or more products are not available");
            }

            // Create the order using AutoMapper
            var order = new Order
            {
                OrderDate = DateTime.Now,
                Status = "Pending",
                UserProfileId = userProfile.Id,
                OrderItems = mapper.Map<List<OrderItem>>(orderDto.Items)
            };

            dbContext.Orders.Add(order);
            await dbContext.SaveChangesAsync();

            // Reload the order with related data
            order = await dbContext.Orders
                .Include(o => o.OrderItems)
                .ThenInclude(oi => oi.Product)
                .Include(o => o.UserProfile)
                .FirstOrDefaultAsync(o => o.Id == order.Id);

            // Map to DTO using AutoMapper
            var createdOrderDto = mapper.Map<OrderDto>(order);

            return Results.Created($"/orders/{order.Id}", createdOrderDto);
        }).RequireAuthorization();

        // Update order status - Only Bakers and Admins
        app.MapPatch("/orders/{id}/status", async (
            int id,
            OrderStatusUpdateDto statusDto,
            TinyTreatsDbContext dbContext,
            IMapper mapper) =>
        {
            var order = await dbContext.Orders.FindAsync(id);
            if (order == null)
            {
                return Results.NotFound();
            }

            // Validate the status
            if (!new[] { "Pending", "Preparing", "Ready", "Delivered" }.Contains(statusDto.Status))
            {
                return Results.BadRequest("Invalid status");
            }

            order.Status = statusDto.Status;

            // If the order is delivered, set the delivery date
            if (statusDto.Status == "Delivered")
            {
                order.DeliveryDate = DateTime.Now;
            }

            await dbContext.SaveChangesAsync();
            return Results.NoContent();
        }).RequireAuthorization(policy => policy.RequireRole("Baker", "Admin"));

        // Cancel an order - Admin can cancel any, Customer can cancel only their own pending orders
        app.MapDelete("/orders/{id}", async (
            int id,
            ClaimsPrincipal user,
            UserManager<IdentityUser> userManager,
            TinyTreatsDbContext dbContext,
            IMapper mapper) =>
        {
            // Get the current user's ID
            var userId = user.FindFirstValue(ClaimTypes.NameIdentifier);
            if (userId == null)
            {
                return Results.Unauthorized();
            }

            // Check if the user is an admin
            bool isAdmin = user.IsInRole("Admin");

            // Get the order
            var order = await dbContext.Orders.FindAsync(id);
            if (order == null)
            {
                return Results.NotFound();
            }

            // Check if the user has permission to cancel this order
            if (!isAdmin)
            {
                // Get the user's profile
                var userProfile = await dbContext.UserProfiles
                    .FirstOrDefaultAsync(up => up.IdentityUserId == userId);

                if (userProfile == null || order.UserProfileId != userProfile.Id)
                {
                    return Results.NotFound(); // Use NotFound instead of Forbidden for security
                }

                // Customers can only cancel pending orders
                if (order.Status != "Pending")
                {
                    return Results.BadRequest("Only pending orders can be canceled");
                }
            }

            // Delete the order items first
            var orderItems = await dbContext.OrderItems
                .Where(oi => oi.OrderId == id)
                .ToListAsync();

            dbContext.OrderItems.RemoveRange(orderItems);

            // Then delete the order
            dbContext.Orders.Remove(order);
            await dbContext.SaveChangesAsync();

            return Results.NoContent();
        }).RequireAuthorization();
    }
}
```

## Understanding the Order Endpoints

Let's break down each endpoint:

### Get All Orders

The get all orders endpoint (`/orders`) returns orders based on the user's role:

```csharp
app.MapGet("/orders", async (
    ClaimsPrincipal user,
    UserManager<IdentityUser> userManager,
    TinyTreatsDbContext dbContext) =>
{
    // Get the current user's ID
    var userId = user.FindFirstValue(ClaimTypes.NameIdentifier);
    if (userId == null)
    {
        return Results.Unauthorized();
    }

    // Check if the user is an admin or baker
    bool isAdmin = user.IsInRole("Admin");
    bool isBaker = user.IsInRole("Baker");

    // Get the user's profile
    var userProfile = await dbContext.UserProfiles
        .FirstOrDefaultAsync(up => up.IdentityUserId == userId);

    if (userProfile == null)
    {
        return Results.NotFound("User profile not found");
    }

    // Query orders based on role
    IQueryable<Order> ordersQuery = dbContext.Orders
        .Include(o => o.OrderItems)
        .ThenInclude(oi => oi.Product)
        .Include(o => o.UserProfile);

    if (!isAdmin && !isBaker)
    {
        // Customers can only see their own orders
        ordersQuery = ordersQuery.Where(o => o.UserProfileId == userProfile.Id);
    }

    var orders = await ordersQuery.ToListAsync();

    // Map to DTOs using AutoMapper
    var orderDtos = mapper.Map<List<OrderDto>>(orders);

    return Results.Ok(orderDtos);
}).RequireAuthorization();
```

This endpoint:
1. Gets the current user's ID from the claims
2. Checks if the user is an admin or baker
3. Gets the user's profile
4. Queries orders based on the user's role:
   - Admins and bakers can see all orders
   - Customers can only see their own orders
5. Maps the orders to DTOs
6. Returns a 200 OK response with the orders
7. Requires authentication

### Get Order by ID

The get order by ID endpoint (`/orders/{id}`) returns a specific order:

```csharp
app.MapGet("/orders/{id}", async (
    int id,
    ClaimsPrincipal user,
    UserManager<IdentityUser> userManager,
    TinyTreatsDbContext dbContext) =>
{
    // Get the current user's ID
    var userId = user.FindFirstValue(ClaimTypes.NameIdentifier);
    if (userId == null)
    {
        return Results.Unauthorized();
    }

    // Check if the user is an admin or baker
    bool isAdmin = user.IsInRole("Admin");
    bool isBaker = user.IsInRole("Baker");

    // Get the order with related data
    var order = await dbContext.Orders
        .Include(o => o.OrderItems)
        .ThenInclude(oi => oi.Product)
        .Include(o => o.UserProfile)
        .FirstOrDefaultAsync(o => o.Id == id);

    if (order == null)
    {
        return Results.NotFound();
    }

    // Check if the user has permission to view this order
    if (!isAdmin && !isBaker)
    {
        // Get the user's profile
        var userProfile = await dbContext.UserProfiles
            .FirstOrDefaultAsync(up => up.IdentityUserId == userId);

        if (userProfile == null || order.UserProfileId != userProfile.Id)
        {
            return Results.NotFound(); // Use NotFound instead of Forbidden for security
        }
    }

    // Map to DTO using AutoMapper
    var orderDto = mapper.Map<OrderDto>(order);

    return Results.Ok(orderDto);
}).RequireAuthorization();
```

This endpoint:
1. Gets the current user's ID from the claims
2. Checks if the user is an admin or baker
3. Gets the order with related data
4. Checks if the user has permission to view the order:
   - Admins and bakers can see any order
   - Customers can only see their own orders
5. Maps the order to a DTO
6. Returns a 200 OK response with the order
7. Requires authentication

### Create Order

The create order endpoint (`/orders`) creates a new order:

```csharp
app.MapPost("/orders", async (
    OrderCreateDto orderDto,
    ClaimsPrincipal user,
    UserManager<IdentityUser> userManager,
    TinyTreatsDbContext dbContext) =>
{
    // Get the current user's ID
    var userId = user.FindFirstValue(ClaimTypes.NameIdentifier);
    if (userId == null)
    {
        return Results.Unauthorized();
    }

    // Get the user's profile
    var userProfile = await dbContext.UserProfiles
        .FirstOrDefaultAsync(up => up.IdentityUserId == userId);

    if (userProfile == null)
    {
        return Results.NotFound("User profile not found");
    }

    // Validate that all products exist and are available
    var productIds = orderDto.Items.Select(i => i.ProductId).ToList();
    var products = await dbContext.Products
        .Where(p => productIds.Contains(p.Id) && p.IsAvailable)
        .ToDictionaryAsync(p => p.Id, p => p);

    // Check if all products were found and are available
    if (products.Count != productIds.Count)
    {
        return Results.BadRequest("One or more products are not available");
    }

    // Create the order
    var order = new Order
    {
        OrderDate = DateTime.Now,
        Status = "Pending",
        UserProfileId = userProfile.Id,
        OrderItems = orderDto.Items.Select(i => new OrderItem
        {
            ProductId = i.ProductId,
            Quantity = i.Quantity
        }).ToList()
    };

    dbContext.Orders.Add(order);
    await dbContext.SaveChangesAsync();

    // Reload the order with related data
    order = await dbContext.Orders
        .Include(o => o.OrderItems)
        .ThenInclude(oi => oi.Product)
        .Include(o => o.UserProfile)
        .FirstOrDefaultAsync(o => o.Id == order.Id);

    // Map to DTO
    var createdOrderDto = new OrderDto
    {
        Id = order.Id,
        OrderDate = order.OrderDate,
        DeliveryDate = order.DeliveryDate,
        Status = order.Status,
        UserProfileId = order.UserProfileId,
        CustomerName = $"{order.UserProfile.FirstName} {order.UserProfile.LastName}",
        Items = order.OrderItems.Select(oi => new OrderItemDto
        {
            ProductId = oi.ProductId,
            ProductName = oi.Product.Name,
            ProductPrice = oi.Product.Price,
            Quantity = oi.Quantity,
            Subtotal = oi.Quantity * oi.Product.Price
        }).ToList(),
        TotalAmount = order.TotalAmount
    };

    return Results.Created($"/orders/{order.Id}", createdOrderDto);
}).RequireAuthorization();
```

This endpoint:
1. Gets the current user's ID from the claims
2. Gets the user's profile
3. Validates that all products exist and are available
4. Creates a new order with the provided items
5. Saves the order to the database
6. Reloads the order with related data
7. Maps the order to a DTO
8. Returns a 201 Created response with the created order
9. Requires authentication

### Update Order Status

The update order status endpoint (`/orders/{id}/status`) updates the status of an order:

```csharp
app.MapPatch("/orders/{id}/status", async (
    int id,
    OrderStatusUpdateDto statusDto,
    TinyTreatsDbContext dbContext) =>
{
    var order = await dbContext.Orders.FindAsync(id);
    if (order == null)
    {
        return Results.NotFound();
    }

    // Validate the status
    if (!new[] { "Pending", "Preparing", "Ready", "Delivered" }.Contains(statusDto.Status))
    {
        return Results.BadRequest("Invalid status");
    }

    order.Status = statusDto.Status;

    // If the order is delivered, set the delivery date
    if (statusDto.Status == "Delivered")
    {
        order.DeliveryDate = DateTime.Now;
    }

    await dbContext.SaveChangesAsync();
    return Results.NoContent();
}).RequireAuthorization(policy => policy.RequireRole("Baker", "Admin"));
```

This endpoint:
1. Gets the order by ID
2. Validates the status
3. Updates the order status
4. Sets the delivery date if the status is "Delivered"
5. Saves the changes to the database
6. Returns a 204 No Content response
7. Requires the "Baker" or "Admin" role

### Cancel Order

The cancel order endpoint (`/orders/{id}`) cancels an order:

```csharp
app.MapDelete("/orders/{id}", async (
    int id,
    ClaimsPrincipal user,
    UserManager<IdentityUser> userManager,
    TinyTreatsDbContext dbContext) =>
{
    // Get the current user's ID
    var userId = user.FindFirstValue(ClaimTypes.NameIdentifier);
    if (userId == null)
    {
        return Results.Unauthorized();
    }

    // Check if the user is an admin
    bool isAdmin = user.IsInRole("Admin");

    // Get the order
    var order = await dbContext.Orders.FindAsync(id);
    if (order == null)
    {
        return Results.NotFound();
    }

    // Check if the user has permission to cancel this order
    if (!isAdmin)
    {
        // Get the user's profile
        var userProfile = await dbContext.UserProfiles
            .FirstOrDefaultAsync(up => up.IdentityUserId == userId);

        if (userProfile == null || order.UserProfileId != userProfile.Id)
        {
            return Results.NotFound(); // Use NotFound instead of Forbidden for security
        }

        // Customers can only cancel pending orders
        if (order.Status != "Pending")
        {
            return Results.BadRequest("Only pending orders can be canceled");
        }
    }

    // Delete the order items first
    var orderItems = await dbContext.OrderItems
        .Where(oi => oi.OrderId == id)
        .ToListAsync();

    dbContext.OrderItems.RemoveRange(orderItems);

    // Then delete the order
    dbContext.Orders.Remove(order);
    await dbContext.SaveChangesAsync();

    return Results.NoContent();
}).RequireAuthorization();
```

This endpoint:
1. Gets the current user's ID from the claims
2. Checks if the user is an admin
3. Gets the order by ID
4. Checks if the user has permission to cancel the order:
   - Admins can cancel any order
   - Customers can only cancel their own pending orders
5. Deletes the order items and the order
6. Returns a 204 No Content response
7. Requires authentication

## Authorization Patterns

Notice the different authorization patterns we're using:

1. **Basic authentication**: Requires the user to be authenticated
   ```csharp
   app.MapPost("/orders", ...).RequireAuthorization();
   ```

2. **Role-based authorization**: Requires the user to have specific roles
   ```csharp
   app.MapPatch("/orders/{id}/status", ...).RequireAuthorization(policy => policy.RequireRole("Baker", "Admin"));
   ```

3. **Custom authorization logic**: Checks permissions in the endpoint handler
   ```csharp
   // Check if the user has permission to view this order
   if (!isAdmin && !isBaker)
   {
       // Get the user's profile
       var userProfile = await dbContext.UserProfiles
           .FirstOrDefaultAsync(up => up.IdentityUserId == userId);

       if (userProfile == null || order.UserProfileId != userProfile.Id)
       {
           return Results.NotFound(); // Use NotFound instead of Forbidden for security
       }
   }
   ```

## Testing Order Management

You can test these endpoints using a tool like Yaak:

1. Login as a customer:
   ```http
   POST /auth/login
   Content-Type: application/json

   {
     "email": "customer1@example.com",
     "password": "Customer123!"
   }
   ```

2. Create a new order:
   ```http
   POST /orders
   Content-Type: application/json

   {
     "items": [
       {
         "productId": 1,
         "quantity": 2
       },
       {
         "productId": 3,
         "quantity": 1
       }
     ]
   }
   ```

3. Get all orders:
   ```http
   GET /orders
   ```

4. Get a specific order:
   ```http
   GET /orders/1
   ```

5. Login as a baker:
   ```http
   POST /auth/login
   Content-Type: application/json

   {
     "email": "baker1@tinytreats.com",
     "password": "Baker123!"
   }
   ```

6. Update an order status:
   ```http
   PATCH /orders/1/status
   Content-Type: application/json

   {
     "status": "Preparing"
   }
   ```

7. Login as an admin:
   ```http
   POST /auth/login
   Content-Type: application/json

   {
     "email": "admin@tinytreats.com",
     "password": "Admin123!"
   }
   ```

8. Cancel an order:
   ```http
   DELETE /orders/1
   ```

## Summary

In this chapter, we've implemented order management endpoints for our TinyTreats application:

- Get all orders endpoint for viewing orders based on the user's role
- Get order by ID endpoint for viewing a specific order
- Create order endpoint for placing new orders
- Update order status endpoint for updating the status of an order
- Cancel order endpoint for canceling orders

These endpoints provide a complete order management system for our bakery application, with appropriate authorization checks to ensure that users can only access and modify data they're allowed to.

In the next chapter, we'll implement the `Program.cs` file to tie everything together and provide instructions for creating migrations and updating the database.

[Next: Program.cs and Database Migrations](./tinytreats-program.md)