# Securing API Endpoints

In this chapter, we'll explore more advanced techniques for securing API endpoints in our TinyTreats application using Minimal API. We'll focus on implementing proper authorization checks and understanding how authorization works in the Minimal API approach, while maintaining our organized endpoint structure.

## Understanding Authorization in Minimal API

In Minimal API, we secure endpoints using the `.RequireAuthorization()` method. Let's explore different ways to use it:

### Basic Authorization

The simplest form requires a user to be authenticated, but doesn't check for any specific roles:

```csharp
// In AuthEndpoints.cs or any other endpoint class
app.MapGet("/secure-data", () =>
{
    // Only authenticated users can access this
    return Results.Ok("This is secure data");
}).RequireAuthorization();
```

### Role-Based Authorization

We can restrict access to specific roles:

```csharp
// In OrderEndpoints.cs
app.MapGet("/admin-data", () =>
{
    // Only users in the Admin role can access this
    return Results.Ok("This is admin-only data");
}).RequireAuthorization(policy => policy.RequireRole("Admin"));
```

We can also allow multiple roles:

```csharp
// In OrderEndpoints.cs
app.MapGet("/staff-data", () =>
{
    // Users in either Admin OR Baker role can access this
    return Results.Ok("This is staff-only data");
}).RequireAuthorization(policy => policy.RequireRole("Admin", "Baker"));
```

### Policy-Based Authorization

For more complex authorization requirements, we can define and use policies:

```csharp
// In Program.cs, during service configuration
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("BakeryStaff", policy =>
        policy.RequireRole("Admin", "Baker"));

    options.AddPolicy("SeniorStaff", policy =>
        policy.RequireRole("Admin"));
});

// Using the policy in an endpoint (in OrderEndpoints.cs)
app.MapGet("/staff-data", () =>
{
    // Only users that satisfy the "BakeryStaff" policy can access this
    return Results.Ok("This is staff-only data");
}).RequireAuthorization("BakeryStaff");
```

## Applying Authorization at Different Levels

In Minimal API, we can apply authorization in different ways:

### Group-Level Authorization

We can apply authorization to a group of endpoints:

```csharp
// In OrderEndpoints.cs
public static void MapOrderEndpoints(this WebApplication app)
{
    var adminGroup = app.MapGroup("/admin")
        .RequireAuthorization(policy => policy.RequireRole("Admin"));

    // All endpoints in this group require Admin role
    adminGroup.MapGet("/data", () => Results.Ok("Admin data"));
    adminGroup.MapPost("/data", () => Results.Ok("Created admin data"));

    // Other endpoints...
}
```

### Endpoint-Level Authorization

Each endpoint can have its own authorization requirements:

```csharp
// In various endpoint classes
// Public endpoint - no authorization
app.MapGet("/public-data", () =>
    Results.Ok("This is public data"));

// User-only endpoint - requires authentication
app.MapGet("/user-data", () =>
    Results.Ok("This is user-only data"))
    .RequireAuthorization();

// Admin-only endpoint - requires Admin role
app.MapGet("/admin-data", () =>
    Results.Ok("This is admin-only data"))
    .RequireAuthorization(policy => policy.RequireRole("Admin"));

// Baker-only endpoint - requires Baker role
app.MapGet("/baker-data", () =>
    Results.Ok("This is baker-only data"))
    .RequireAuthorization(policy => policy.RequireRole("Baker"));
```

## Creating an Order System with Authorization

Let's create a simple order system for our TinyTreats application to demonstrate authorization in a real-world scenario. We've already defined the Order model and endpoints in previous chapters, but let's review how we've secured them:

### Order Model

```csharp
// Models/Order.cs
using System.ComponentModel.DataAnnotations;

namespace TinyTreats.Models;

public class Order
{
    public int Id { get; set; }

    [Required]
    public DateTime OrderDate { get; set; }

    public DateTime? DeliveryDate { get; set; }

    [Required]
    public string Status { get; set; } // "Pending", "Preparing", "Ready", "Delivered"

    [Required]
    public int UserProfileId { get; set; }
    public UserProfile UserProfile { get; set; }

    public List<OrderItem> OrderItems { get; set; }

    public decimal TotalAmount => OrderItems?.Sum(item => item.Quantity * item.Product.Price) ?? 0;
}

public class OrderItem
{
    public int Id { get; set; }

    [Required]
    public int OrderId { get; set; }
    public Order Order { get; set; }

    [Required]
    public int ProductId { get; set; }
    public Product Product { get; set; }

    [Required]
    [Range(1, int.MaxValue)]
    public int Quantity { get; set; }
}
```

### Order Endpoints with Authorization

In our `OrderEndpoints.cs` file, we've implemented various endpoints with different authorization requirements:

```csharp
// Endpoints/OrderEndpoints.cs
public static class OrderEndpoints
{
    // DTOs for order creation
    public class OrderCreateDto
    {
        public List<OrderItemDto> Items { get; set; }
    }

    public class OrderItemDto
    {
        public int ProductId { get; set; }
        public int Quantity { get; set; }
    }

    public static void MapOrderEndpoints(this WebApplication app)
    {
        // Get all orders - Admin can see all, others see only their own
        app.MapGet("/order", async (
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
                .ThenInclude(oi => oi.Product);

            if (!isAdmin)
            {
                // Non-admins can only see their own orders
                ordersQuery = ordersQuery.Where(o => o.UserProfileId == userProfile.Id);
            }

            var orders = await ordersQuery.ToListAsync();
            return Results.Ok(orders);
        }).RequireAuthorization();

        // Other order endpoints...

        // Update order status - Only Bakers and Admins
        app.MapPatch("/order/{id}/status", async (
            int id,
            string newStatus,
            TinyTreatsDbContext dbContext) =>
        {
            var order = await dbContext.Orders.FindAsync(id);
            if (order == null)
            {
                return Results.NotFound();
            }

            // Validate the status
            if (!new[] { "Pending", "Preparing", "Ready", "Delivered" }.Contains(newStatus))
            {
                return Results.BadRequest("Invalid status");
            }

            order.Status = newStatus;

            // If the order is delivered, set the delivery date
            if (newStatus == "Delivered")
            {
                order.DeliveryDate = DateTime.Now;
            }

            await dbContext.SaveChangesAsync();
            return Results.NoContent();
        }).RequireAuthorization(policy => policy.RequireRole("Baker", "Admin"));
    }
}
```

## Understanding Authorization Checks in Code

In addition to using the `.RequireAuthorization()` method, we can perform authorization checks in our code:

```csharp
// Check if the user is in a role
bool isAdmin = user.IsInRole("Admin");

// Check if the user has a specific claim
string userId = user.FindFirstValue(ClaimTypes.NameIdentifier);

// Conditional authorization logic
if (!isAdmin && order.UserProfileId != userProfile.Id)
{
    return Results.Forbid(); // Return 403 Forbidden
}
```

This allows for more complex authorization rules that can't be expressed with the `.RequireAuthorization()` method alone.

## Best Practices for Securing API Endpoints

Here are some best practices to follow when securing your API endpoints:

1. **Use the right status codes**:
   - 401 Unauthorized: The user is not authenticated
   - 403 Forbidden: The user is authenticated but not authorized
   - 404 Not Found: For security, sometimes it's better to return 404 instead of 403

2. **Apply authorization at multiple levels**:
   - Use `.RequireAuthorization()` for basic checks
   - Implement additional checks in your code for complex rules

3. **Don't trust client input**:
   - Always verify that the user has permission to access or modify the requested resource
   - Check that the user owns the resource or has the appropriate role

4. **Use HTTPS**:
   - Always use HTTPS in production to encrypt data in transit
   - ASP.NET Core can enforce HTTPS with `app.UseHttpsRedirection()`

5. **Implement proper logging**:
   - Log authentication and authorization failures
   - Monitor for suspicious activity

6. **Organize endpoints by functionality**:
   - Group related endpoints in separate files
   - Use extension methods to keep `Program.cs` clean
   - Apply consistent authorization patterns across similar endpoints

## Summary

In this chapter, we've explored advanced techniques for securing API endpoints in our TinyTreats application using Minimal API:

- Using the `.RequireAuthorization()` method at different levels
- Implementing role-based and policy-based authorization
- Performing authorization checks in code
- Creating a real-world order system with proper authorization
- Following best practices for API security
- Organizing endpoints by functionality for better maintainability

With these techniques, you can create secure, robust APIs that protect your data and provide appropriate access to different types of users.

This completes our series on authentication and authorization in ASP.NET Core with Minimal API. You now have the knowledge to implement secure, role-based authentication in your own applications.