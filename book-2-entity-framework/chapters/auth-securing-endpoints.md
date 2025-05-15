# Securing API Endpoints

In this chapter, we'll explore more advanced techniques for securing API endpoints in our TinyTreats application. We'll focus on implementing proper authorization checks and understanding how the `[Authorize]` attribute works.

## Understanding the `[Authorize]` Attribute

The `[Authorize]` attribute is a powerful tool for securing API endpoints. Let's explore its different uses:

### Basic Authorization

The simplest form of the `[Authorize]` attribute requires a user to be authenticated, but doesn't check for any specific roles:

```csharp
[HttpGet]
[Authorize]
public IActionResult GetSecureData()
{
    // Only authenticated users can access this
    return Ok("This is secure data");
}
```

### Role-Based Authorization

We can restrict access to specific roles:

```csharp
[HttpGet]
[Authorize(Roles = "Admin")]
public IActionResult AdminOnlyData()
{
    // Only users in the Admin role can access this
    return Ok("This is admin-only data");
}
```

We can also allow multiple roles:

```csharp
[HttpGet]
[Authorize(Roles = "Admin,Baker")]
public IActionResult StaffOnlyData()
{
    // Users in either Admin OR Baker role can access this
    return Ok("This is staff-only data");
}
```

### Policy-Based Authorization

For more complex authorization requirements, we can use policies:

```csharp
// In Program.cs
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("BakeryStaff", policy =>
        policy.RequireRole("Admin", "Baker"));

    options.AddPolicy("SeniorStaff", policy =>
        policy.RequireRole("Admin"));
});

// In a controller
[HttpGet]
[Authorize(Policy = "BakeryStaff")]
public IActionResult StaffOnlyData()
{
    // Only users that satisfy the "BakeryStaff" policy can access this
    return Ok("This is staff-only data");
}
```

## Applying Authorization at Different Levels

Authorization can be applied at different levels in your application:

### Controller-Level Authorization

You can apply authorization to an entire controller:

```csharp
[ApiController]
[Route("api/[controller]")]
[Authorize(Roles = "Admin")]
public class AdminController : ControllerBase
{
    // All methods in this controller require Admin role

    [HttpGet]
    public IActionResult Get()
    {
        return Ok("Admin data");
    }

    [HttpPost]
    public IActionResult Post()
    {
        return Ok("Created admin data");
    }
}
```

### Method-Level Authorization

You can override controller-level authorization at the method level:

```csharp
[ApiController]
[Route("api/[controller]")]
[Authorize] // Requires authentication for all methods
public class MixedController : ControllerBase
{
    [HttpGet]
    [AllowAnonymous] // Overrides the controller-level [Authorize]
    public IActionResult PublicData()
    {
        return Ok("This is public data");
    }

    [HttpGet("admin")]
    [Authorize(Roles = "Admin")] // Requires Admin role
    public IActionResult AdminData()
    {
        return Ok("This is admin-only data");
    }

    [HttpGet("baker")]
    [Authorize(Roles = "Baker")] // Requires Baker role
    public IActionResult BakerData()
    {
        return Ok("This is baker-only data");
    }
}
```

## Creating an Order System with Authorization

Let's create a simple order system for our TinyTreats application to demonstrate authorization in a real-world scenario:

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

Update the DbContext:

```csharp
// In TinyTreatsDbContext.cs
public DbSet<Order> Orders { get; set; }
public DbSet<OrderItem> OrderItems { get; set; }
```

### Order Controller with Authorization

```csharp
// Controllers/OrderController.cs
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using System.Security.Claims;
using TinyTreats.Data;
using TinyTreats.Models;

namespace TinyTreats.Controllers;

[ApiController]
[Route("api/[controller]")]
[Authorize] // Base authorization - must be logged in
public class OrderController : ControllerBase
{
    private TinyTreatsDbContext _dbContext;

    public OrderController(TinyTreatsDbContext context)
    {
        _dbContext = context;
    }

    // Get all orders - Admin can see all, others see only their own
    [HttpGet]
    public async Task<IActionResult> GetOrders()
    {
        // Get the current user's ID
        var userId = User.FindFirstValue(ClaimTypes.NameIdentifier);
        if (userId == null)
        {
            return Unauthorized();
        }

        // Check if the user is an admin
        bool isAdmin = User.IsInRole("Admin");

        // Get the user's profile
        var userProfile = await _dbContext.UserProfiles
            .FirstOrDefaultAsync(up => up.IdentityUserId == userId);

        if (userProfile == null)
        {
            return NotFound("User profile not found");
        }

        // Query orders based on role
        IQueryable<Order> ordersQuery = _dbContext.Orders
            .Include(o => o.OrderItems)
            .ThenInclude(oi => oi.Product);

        if (!isAdmin)
        {
            // Non-admins can only see their own orders
            ordersQuery = ordersQuery.Where(o => o.UserProfileId == userProfile.Id);
        }

        var orders = await ordersQuery.ToListAsync();
        return Ok(orders);
    }

    // Get a specific order - Admin can see any, others see only their own
    [HttpGet("{id}")]
    public async Task<IActionResult> GetOrder(int id)
    {
        // Get the current user's ID
        var userId = User.FindFirstValue(ClaimTypes.NameIdentifier);
        if (userId == null)
        {
            return Unauthorized();
        }

        // Check if the user is an admin
        bool isAdmin = User.IsInRole("Admin");

        // Get the user's profile
        var userProfile = await _dbContext.UserProfiles
            .FirstOrDefaultAsync(up => up.IdentityUserId == userId);

        if (userProfile == null)
        {
            return NotFound("User profile not found");
        }

        // Get the order with items
        var order = await _dbContext.Orders
            .Include(o => o.OrderItems)
            .ThenInclude(oi => oi.Product)
            .FirstOrDefaultAsync(o => o.Id == id);

        if (order == null)
        {
            return NotFound();
        }

        // Check if the user is authorized to view this order
        if (!isAdmin && order.UserProfileId != userProfile.Id)
        {
            return Forbid();
        }

        return Ok(order);
    }

    // Create a new order - Any authenticated user
    [HttpPost]
    public async Task<IActionResult> CreateOrder(OrderCreateDto orderDto)
    {
        // Get the current user's ID
        var userId = User.FindFirstValue(ClaimTypes.NameIdentifier);
        if (userId == null)
        {
            return Unauthorized();
        }

        // Get the user's profile
        var userProfile = await _dbContext.UserProfiles
            .FirstOrDefaultAsync(up => up.IdentityUserId == userId);

        if (userProfile == null)
        {
            return NotFound("User profile not found");
        }

        // Validate that all products exist
        foreach (var item in orderDto.Items)
        {
            var product = await _dbContext.Products.FindAsync(item.ProductId);
            if (product == null)
            {
                return BadRequest($"Product with ID {item.ProductId} not found");
            }

            if (product.Stock < item.Quantity)
            {
                return BadRequest($"Not enough stock for product {product.Name}");
            }
        }

        // Create the order
        var order = new Order
        {
            OrderDate = DateTime.Now,
            Status = "Pending",
            UserProfileId = userProfile.Id,
            OrderItems = orderDto.Items.Select(item => new OrderItem
            {
                ProductId = item.ProductId,
                Quantity = item.Quantity
            }).ToList()
        };

        // Update product stock
        foreach (var item in order.OrderItems)
        {
            var product = await _dbContext.Products.FindAsync(item.ProductId);
            product.Stock -= item.Quantity;
        }

        _dbContext.Orders.Add(order);
        await _dbContext.SaveChangesAsync();

        return CreatedAtAction(nameof(GetOrder), new { id = order.Id }, order);
    }

    // Update order status - Only Bakers and Admins
    [HttpPatch("{id}/status")]
    [Authorize(Roles = "Baker,Admin")]
    public async Task<IActionResult> UpdateOrderStatus(int id, [FromBody] string newStatus)
    {
        var order = await _dbContext.Orders.FindAsync(id);
        if (order == null)
        {
            return NotFound();
        }

        // Validate the status
        if (!new[] { "Pending", "Preparing", "Ready", "Delivered" }.Contains(newStatus))
        {
            return BadRequest("Invalid status");
        }

        order.Status = newStatus;

        // If the order is delivered, set the delivery date
        if (newStatus == "Delivered")
        {
            order.DeliveryDate = DateTime.Now;
        }

        await _dbContext.SaveChangesAsync();
        return NoContent();
    }

    // Cancel an order - Admin can cancel any, others only their own pending orders
    [HttpDelete("{id}")]
    public async Task<IActionResult> CancelOrder(int id)
    {
        // Get the current user's ID
        var userId = User.FindFirstValue(ClaimTypes.NameIdentifier);
        if (userId == null)
        {
            return Unauthorized();
        }

        // Check if the user is an admin
        bool isAdmin = User.IsInRole("Admin");

        // Get the user's profile
        var userProfile = await _dbContext.UserProfiles
            .FirstOrDefaultAsync(up => up.IdentityUserId == userId);

        if (userProfile == null)
        {
            return NotFound("User profile not found");
        }

        // Get the order
        var order = await _dbContext.Orders
            .Include(o => o.OrderItems)
            .FirstOrDefaultAsync(o => o.Id == id);

        if (order == null)
        {
            return NotFound();
        }

        // Check if the user is authorized to cancel this order
        if (!isAdmin)
        {
            // Non-admins can only cancel their own orders
            if (order.UserProfileId != userProfile.Id)
            {
                return Forbid();
            }

            // Non-admins can only cancel pending orders
            if (order.Status != "Pending")
            {
                return BadRequest("Cannot cancel an order that is already being prepared");
            }
        }

        // Return items to stock
        foreach (var item in order.OrderItems)
        {
            var product = await _dbContext.Products.FindAsync(item.ProductId);
            if (product != null)
            {
                product.Stock += item.Quantity;
            }
        }

        // Remove the order
        _dbContext.OrderItems.RemoveRange(order.OrderItems);
        _dbContext.Orders.Remove(order);
        await _dbContext.SaveChangesAsync();

        return NoContent();
    }
}

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
```

Create a migration for the new Order models:

```bash
dotnet ef migrations add AddOrders
dotnet ef database update
```

## Understanding Authorization Checks in Code

In addition to using the `[Authorize]` attribute, we can perform authorization checks in our code:

```csharp
// Check if the user is in a role
bool isAdmin = User.IsInRole("Admin");

// Check if the user has a specific claim
string userId = User.FindFirstValue(ClaimTypes.NameIdentifier);

// Conditional authorization logic
if (!isAdmin && order.UserProfileId != userProfile.Id)
{
    return Forbid(); // Return 403 Forbidden
}
```

This allows for more complex authorization rules that can't be expressed with attributes alone.

## Best Practices for Securing API Endpoints

Here are some best practices to follow when securing your API endpoints:

1. **Use the right status codes**:
   - 401 Unauthorized: The user is not authenticated
   - 403 Forbidden: The user is authenticated but not authorized
   - 404 Not Found: For security, sometimes it's better to return 404 instead of 403

2. **Apply authorization at multiple levels**:
   - Use `[Authorize]` attributes for basic checks
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

## Summary

In this chapter, we've explored advanced techniques for securing API endpoints in our TinyTreats application:

- Using the `[Authorize]` attribute at different levels
- Implementing role-based and policy-based authorization
- Performing authorization checks in code
- Creating a real-world order system with proper authorization
- Following best practices for API security

With these techniques, you can create secure, robust APIs that protect your data and provide appropriate access to different types of users.

This completes our series on authentication and authorization in ASP.NET Core. You now have the knowledge to implement secure, role-based authentication in your own applications.

[Back to Introduction](./auth-introduction.md)