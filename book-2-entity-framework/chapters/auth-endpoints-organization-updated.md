# Organizing Authentication Endpoints

In this chapter, we'll learn how to organize our authentication endpoints using the same pattern we used for our resource endpoints. This approach helps keep our `Program.cs` file clean and makes our authentication code more maintainable.

## Understanding the Endpoints Directory Structure

In our TinyTreats application, we'll create an `Endpoints` directory with separate files for different types of endpoints:

```
TinyTreats/
├── DTOs/
│   ├── AuthDtos.cs
│   ├── OrderDtos.cs
│   └── RoleDtos.cs
├── Endpoints/
│   ├── AuthEndpoints.cs
│   └── OrderEndpoints.cs
├── Models/
│   ├── UserProfile.cs
│   └── Order.cs
├── Program.cs
└── ...
```

This structure makes it easy to find and modify endpoints related to specific functionality, while keeping DTOs in their own dedicated directory.

## Creating the DTO Classes

First, let's create our DTO classes in a separate directory. Create a new file called `DTOs/AuthDtos.cs`:

```csharp
// DTOs/AuthDtos.cs
namespace TinyTreats.DTOs;

public class RegistrationDto
{
    public string Email { get; set; }
    public string Password { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Address { get; set; }
}

public class LoginDto
{
    public string Email { get; set; }
    public string Password { get; set; }
}
```

Now, let's create a file for order-related DTOs. Create a new file called `DTOs/OrderDtos.cs`:

```csharp
// DTOs/OrderDtos.cs
namespace TinyTreats.DTOs;

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

## Creating the AuthEndpoints Class

Now, let's create our authentication endpoints class. Create a new file called `Endpoints/AuthEndpoints.cs`:

```csharp
using Microsoft.AspNetCore.Identity;
using System.Security.Claims;
using TinyTreats.Data;
using TinyTreats.DTOs;
using TinyTreats.Models;

namespace TinyTreats.Endpoints;

public static class AuthEndpoints
{
    public static void MapAuthEndpoints(this WebApplication app)
    {
        // Registration endpoint
        app.MapPost("/api/auth/register", async (
            RegistrationDto registration,
            UserManager<IdentityUser> userManager,
            SignInManager<IdentityUser> signInManager,
            TinyTreatsDbContext dbContext) =>
        {
            // Create a new Identity user
            var user = new IdentityUser
            {
                UserName = registration.Email,
                Email = registration.Email
            };

            // Try to create the user with the provided password
            var result = await userManager.CreateAsync(user, registration.Password);

            if (result.Succeeded)
            {
                // Create a UserProfile for the new user
                dbContext.UserProfiles.Add(new UserProfile
                {
                    FirstName = registration.FirstName,
                    LastName = registration.LastName,
                    Address = registration.Address,
                    IdentityUserId = user.Id
                });
                await dbContext.SaveChangesAsync();

                // Log the user in
                await signInManager.SignInAsync(user, isPersistent: false);
                return Results.Ok();
            }

            // If we get here, something went wrong
            return Results.BadRequest(result.Errors);
        });

        // Login endpoint
        app.MapPost("/api/auth/login", async (
            LoginDto login,
            UserManager<IdentityUser> userManager,
            SignInManager<IdentityUser> signInManager) =>
        {
            // Find the user by email
            var user = await userManager.FindByEmailAsync(login.Email);

            if (user == null)
            {
                return Results.Unauthorized();
            }

            // Verify the password
            var result = await signInManager.CheckPasswordSignInAsync(user, login.Password, false);

            if (result.Succeeded)
            {
                // Sign in the user
                await signInManager.SignInAsync(user, isPersistent: false);
                return Results.Ok();
            }

            return Results.Unauthorized();
        });

        // Logout endpoint
        app.MapPost("/api/auth/logout", async (SignInManager<IdentityUser> signInManager) =>
        {
            await signInManager.SignOutAsync();
            return Results.Ok();
        });

        // Get current user info
        app.MapGet("/api/auth/me", (ClaimsPrincipal user, TinyTreatsDbContext dbContext) =>
        {
            // Get the user ID from the claims
            var identityUserId = user.FindFirstValue(ClaimTypes.NameIdentifier);

            if (identityUserId == null)
            {
                return Results.Unauthorized();
            }

            // Find the user profile
            var profile = dbContext.UserProfiles
                .FirstOrDefault(up => up.IdentityUserId == identityUserId);

            if (profile == null)
            {
                return Results.NotFound();
            }

            // Return the user profile
            return Results.Ok(new
            {
                profile.Id,
                profile.FirstName,
                profile.LastName,
                profile.Address,
                Email = user.FindFirstValue(ClaimTypes.Email)
            });
        }).RequireAuthorization(); // This is a shorthand for requiring authentication
    }
}
```

## Creating the OrderEndpoints Class

Now, let's create a class for order-related endpoints. Create a new file called `Endpoints/OrderEndpoints.cs`:

```csharp
using Microsoft.AspNetCore.Identity;
using Microsoft.EntityFrameworkCore;
using System.Security.Claims;
using TinyTreats.Data;
using TinyTreats.DTOs;
using TinyTreats.Models;

namespace TinyTreats.Endpoints;

public static class OrderEndpoints
{
    public static void MapOrderEndpoints(this WebApplication app)
    {
        // Get all orders - Admin can see all, others see only their own
        app.MapGet("/api/order", async (
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

        // Get a specific order - Admin can see any, others see only their own
        app.MapGet("/api/order/{id}", async (
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

            // Get the user's profile
            var userProfile = await dbContext.UserProfiles
                .FirstOrDefaultAsync(up => up.IdentityUserId == userId);

            if (userProfile == null)
            {
                return Results.NotFound("User profile not found");
            }

            // Get the order with items
            var order = await dbContext.Orders
                .Include(o => o.OrderItems)
                .ThenInclude(oi => oi.Product)
                .FirstOrDefaultAsync(o => o.Id == id);

            if (order == null)
            {
                return Results.NotFound();
            }

            // Check if the user is authorized to view this order
            if (!isAdmin && order.UserProfileId != userProfile.Id)
            {
                return Results.Forbid();
            }

            return Results.Ok(order);
        }).RequireAuthorization();

        // Create a new order - Any authenticated user
        app.MapPost("/api/order", async (
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

            // Validate that all products exist
            foreach (var item in orderDto.Items)
            {
                var product = await dbContext.Products.FindAsync(item.ProductId);
                if (product == null)
                {
                    return Results.BadRequest($"Product with ID {item.ProductId} not found");
                }

                if (product.Stock < item.Quantity)
                {
                    return Results.BadRequest($"Not enough stock for product {product.Name}");
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
                var product = await dbContext.Products.FindAsync(item.ProductId);
                product.Stock -= item.Quantity;
            }

            dbContext.Orders.Add(order);
            await dbContext.SaveChangesAsync();

            return Results.Created($"/api/order/{order.Id}", order);
        }).RequireAuthorization();

        // Update order status - Only Bakers and Admins
        app.MapPatch("/api/order/{id}/status", async (
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

        // Cancel an order - Admin can cancel any, others only their own pending orders
        app.MapDelete("/api/order/{id}", async (
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

            // Get the user's profile
            var userProfile = await dbContext.UserProfiles
                .FirstOrDefaultAsync(up => up.IdentityUserId == userId);

            if (userProfile == null)
            {
                return Results.NotFound("User profile not found");
            }

            // Get the order
            var order = await dbContext.Orders
                .Include(o => o.OrderItems)
                .FirstOrDefaultAsync(o => o.Id == id);

            if (order == null)
            {
                return Results.NotFound();
            }

            // Check if the user is authorized to cancel this order
            if (!isAdmin)
            {
                // Non-admins can only cancel their own orders
                if (order.UserProfileId != userProfile.Id)
                {
                    return Results.Forbid();
                }

                // Non-admins can only cancel pending orders
                if (order.Status != "Pending")
                {
                    return Results.BadRequest("Cannot cancel an order that is already being prepared");
                }
            }

            // Return items to stock
            foreach (var item in order.OrderItems)
            {
                var product = await dbContext.Products.FindAsync(item.ProductId);
                if (product != null)
                {
                    product.Stock += item.Quantity;
                }
            }

            // Remove the order
            dbContext.OrderItems.RemoveRange(order.OrderItems);
            dbContext.Orders.Remove(order);
            await dbContext.SaveChangesAsync();

            return Results.NoContent();
        }).RequireAuthorization();
    }
}
```

## Updating Program.cs

With our endpoint classes in place, we can now update `Program.cs` to use them. The updated file should look like this:

```csharp
using Microsoft.AspNetCore.Identity;
using Microsoft.EntityFrameworkCore;
using TinyTreats.Data;
using TinyTreats.Endpoints;

var builder = WebApplication.CreateBuilder(args);

// Configure database
builder.Services.AddDbContext<TinyTreatsDbContext>(options =>
    options.UseNpgsql(builder.Configuration["TinyTreatsDbConnectionString"]));

// Configure Identity
builder.Services.AddIdentityCore<IdentityUser>(options =>
{
    // For development, we can use simple password requirements
    options.Password.RequireDigit = true;
    options.Password.RequiredLength = 6;
    options.Password.RequireLowercase = true;
    options.Password.RequireNonAlphanumeric = false;
    options.Password.RequireUppercase = true;
})
.AddRoles<IdentityRole>() // Add role management
.AddEntityFrameworkStores<TinyTreatsDbContext>() // Use our DbContext
.AddSignInManager(); // Add SignInManager

// Configure authentication with cookies
builder.Services.AddAuthentication("Identity.Application")
    .AddCookie("Identity.Application", options =>
    {
        options.Cookie.Name = "TinyTreatsAuth";
        options.ExpireTimeSpan = TimeSpan.FromHours(8);
    });

builder.Services.AddAuthorization();
builder.Services.AddEndpointsApiExplorer();

var app = builder.Build();

// Ensure that HTTPS protocol is used
app.UseHttpsRedirection();

// Add authentication middleware
app.UseAuthentication();
app.UseAuthorization();

// Map API endpoints by functionality
app.MapAuthEndpoints();
app.MapOrderEndpoints();

app.Run();
```

## Benefits of This Approach

Organizing endpoints by functionality and separating DTOs into their own directory offers several benefits:

1. **Better organization** - Endpoints are grouped by functionality, making them easier to find and modify.
2. **Improved maintainability** - Each resource's endpoints are contained in a single file, reducing the complexity of `Program.cs`.
3. **Scalability** - As your application grows, you can add new endpoint classes without cluttering `Program.cs`.
4. **Testability** - Endpoint classes can be tested independently.
5. **Reusability** - Extension methods can be reused across different applications.
6. **Separation of concerns** - DTOs are separated from endpoint logic, following the Single Responsibility Principle.
7. **DTO reusability** - DTOs can be easily reused across different endpoints.

## Conclusion

In this chapter, we've learned how to organize our authentication endpoints using extension methods and how to properly separate DTOs into their own directory. This approach helps keep our code clean, maintainable, and scalable as our application grows.

In the next chapters, we'll continue building our API by implementing specific endpoints for authentication and authorization.

## 🔍 Additional Materials

- [Extension Methods in C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/extension-methods)
- [ASP.NET Core Minimal APIs](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis)
- [API Design Best Practices](https://docs.microsoft.com/en-us/azure/architecture/best-practices/api-design)
- [Data Transfer Objects (DTOs) in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/web-api/overview/data/using-web-api-with-entity-framework/part-5)