# User Roles and Authorization

In this chapter, we'll expand our TinyTreats application to include role-based authorization. This will allow us to control what different types of users can do in our application.

## Understanding Roles

Roles are a way to group users with similar permissions. For example, in our bakery management system, we might have:

- **Administrators**: Can manage users, products, and all aspects of the system
- **Bakers**: Can add and update recipes and products
- **Customers**: Can view products and place orders

By assigning users to these roles, we can easily control what parts of the application they can access.

## Adding Roles to Our Application

Let's update our application to support roles. We'll need to:

1. Seed roles in the database
2. Add methods to assign roles to users
3. Add role-based authorization to our controllers

### Seeding Roles in the Database

First, let's update our `TinyTreatsDbContext` to seed some default roles:

```csharp
// Data/TinyTreatsDbContext.cs (updated OnModelCreating method)
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    base.OnModelCreating(modelBuilder);

    // Seed roles
    modelBuilder.Entity<IdentityRole>().HasData(
        new IdentityRole
        {
            Id = "fab4fac1-c546-41de-aebc-a14da6895711",
            Name = "Admin",
            NormalizedName = "ADMIN"
        },
        new IdentityRole
        {
            Id = "c7b013f0-5201-4317-abd8-c211f91b7330",
            Name = "Baker",
            NormalizedName = "BAKER"
        },
        new IdentityRole
        {
            Id = "2c5e174e-3b0e-446f-86af-483d56fd7210",
            Name = "Customer",
            NormalizedName = "CUSTOMER"
        }
    );

    // Seed an admin user (same as before)
    modelBuilder.Entity<IdentityUser>().HasData(new IdentityUser
    {
        Id = "dbc40bc6-0829-4ac5-a3ed-180f5e916a5f",
        UserName = "admin@tinytreats.com",
        Email = "admin@tinytreats.com",
        NormalizedEmail = "ADMIN@TINYTREATS.COM",
        NormalizedUserName = "ADMIN@TINYTREATS.COM",
        EmailConfirmed = true,
        PasswordHash = new PasswordHasher<IdentityUser>().HashPassword(null, "Admin123!")
    });

    // Seed a user profile for the admin (same as before)
    modelBuilder.Entity<UserProfile>().HasData(new UserProfile
    {
        Id = 1,
        IdentityUserId = "dbc40bc6-0829-4ac5-a3ed-180f5e916a5f",
        FirstName = "Admin",
        LastName = "User",
        Address = "123 Bakery Lane"
    });

    // Assign the admin user to the Admin role
    modelBuilder.Entity<IdentityUserRole<string>>().HasData(new IdentityUserRole<string>
    {
        RoleId = "fab4fac1-c546-41de-aebc-a14da6895711", // Admin role ID
        UserId = "dbc40bc6-0829-4ac5-a3ed-180f5e916a5f"  // Admin user ID
    });
}
```

After updating the database context, we need to create a new migration and update the database:

```bash
dotnet ef migrations add AddRoles
dotnet ef database update
```

### Updating the Auth Controller

Now, let's update our `AuthController` to support roles. We'll add:

1. A method to assign a default role during registration
2. Role information in the "me" endpoint

```csharp
// Controllers/AuthController.cs (updated methods)

// Registration endpoint (updated)
[HttpPost("register")]
public async Task<IActionResult> Register([FromBody] RegistrationDto registration)
{
    // Create a new Identity user
    var user = new IdentityUser
    {
        UserName = registration.Email,
        Email = registration.Email
    };

    // Try to create the user with the provided password
    var result = await _userManager.CreateAsync(user, registration.Password);

    if (result.Succeeded)
    {
        // Assign the Customer role by default
        await _userManager.AddToRoleAsync(user, "Customer");

        // Create a UserProfile for the new user
        _dbContext.UserProfiles.Add(new UserProfile
        {
            FirstName = registration.FirstName,
            LastName = registration.LastName,
            Address = registration.Address,
            IdentityUserId = user.Id
        });
        await _dbContext.SaveChangesAsync();

        // Log the user in
        await _signInManager.SignInAsync(user, isPersistent: false);
        return Ok();
    }

    // If we get here, something went wrong
    return BadRequest(result.Errors);
}

// Get current user info (updated)
[HttpGet("me")]
public async Task<IActionResult> Me()
{
    // Get the user ID from the claims
    var identityUserId = User.FindFirstValue(ClaimTypes.NameIdentifier);

    if (identityUserId == null)
    {
        return Unauthorized();
    }

    // Find the user profile
    var profile = _dbContext.UserProfiles
        .FirstOrDefault(up => up.IdentityUserId == identityUserId);

    if (profile == null)
    {
        return NotFound();
    }

    // Get the user's roles
    var user = await _userManager.FindByIdAsync(identityUserId);
    var roles = await _userManager.GetRolesAsync(user);

    // Return the user profile with roles
    return Ok(new
    {
        profile.Id,
        profile.FirstName,
        profile.LastName,
        profile.Address,
        Email = User.FindFirstValue(ClaimTypes.Email),
        Roles = roles
    });
}
```

### Adding Role Management

Let's add methods to promote and demote users between roles. We'll create a new controller for user management:

```csharp
// Controllers/UserProfileController.cs
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using System.Security.Claims;
using TinyTreats.Data;
using TinyTreats.Models;

namespace TinyTreats.Controllers;

[ApiController]
[Route("api/[controller]")]
public class UserProfileController : ControllerBase
{
    private TinyTreatsDbContext _dbContext;
    private UserManager<IdentityUser> _userManager;

    public UserProfileController(
        TinyTreatsDbContext context,
        UserManager<IdentityUser> userManager)
    {
        _dbContext = context;
        _userManager = userManager;
    }

    // Get all user profiles (Admin only)
    [HttpGet]
    [Authorize(Roles = "Admin")]
    public async Task<IActionResult> GetAllUsers()
    {
        var profiles = await _dbContext.UserProfiles
            .Include(up => up.IdentityUser)
            .Select(up => new
            {
                up.Id,
                up.FirstName,
                up.LastName,
                up.Address,
                Email = up.IdentityUser.Email,
                IdentityUserId = up.IdentityUserId,
                Roles = _userManager.GetRolesAsync(up.IdentityUser).Result
            })
            .ToListAsync();

        return Ok(profiles);
    }

    // Promote a user to Baker
    [HttpPost("promote/{id}")]
    [Authorize(Roles = "Admin")]
    public async Task<IActionResult> PromoteToBaker(string id)
    {
        var user = await _userManager.FindByIdAsync(id);
        if (user == null)
        {
            return NotFound();
        }

        // Check if user is already a Baker
        if (await _userManager.IsInRoleAsync(user, "Baker"))
        {
            return BadRequest("User is already a Baker");
        }

        // Add Baker role
        await _userManager.AddToRoleAsync(user, "Baker");
        return NoContent();
    }

    // Demote a user from Baker to Customer
    [HttpPost("demote/{id}")]
    [Authorize(Roles = "Admin")]
    public async Task<IActionResult> DemoteFromBaker(string id)
    {
        var user = await _userManager.FindByIdAsync(id);
        if (user == null)
        {
            return NotFound();
        }

        // Check if user is a Baker
        if (!await _userManager.IsInRoleAsync(user, "Baker"))
        {
            return BadRequest("User is not a Baker");
        }

        // Remove Baker role
        await _userManager.RemoveFromRoleAsync(user, "Baker");
        return NoContent();
    }
}
```

## Creating a Product Controller with Role-Based Authorization

Now, let's create a simple `Product` model and controller to demonstrate role-based authorization:

```csharp
// Models/Product.cs
namespace TinyTreats.Models;

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
    public decimal Price { get; set; }
    public int Stock { get; set; }
}
```

Add the `Products` DbSet to our context:

```csharp
// In TinyTreatsDbContext.cs
public DbSet<Product> Products { get; set; }
```

Now, let's create a controller with different authorization levels:

```csharp
// Controllers/ProductController.cs
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using TinyTreats.Data;
using TinyTreats.Models;

namespace TinyTreats.Controllers;

[ApiController]
[Route("api/[controller]")]
public class ProductController : ControllerBase
{
    private TinyTreatsDbContext _dbContext;

    public ProductController(TinyTreatsDbContext context)
    {
        _dbContext = context;
    }

    // Get all products - anyone can access
    [HttpGet]
    public async Task<IActionResult> GetAll()
    {
        return Ok(await _dbContext.Products.ToListAsync());
    }

    // Get a single product - anyone can access
    [HttpGet("{id}")]
    public async Task<IActionResult> GetById(int id)
    {
        var product = await _dbContext.Products.FindAsync(id);
        if (product == null)
        {
            return NotFound();
        }
        return Ok(product);
    }

    // Create a product - Bakers and Admins only
    [HttpPost]
    [Authorize(Roles = "Baker,Admin")]
    public async Task<IActionResult> Create(Product product)
    {
        _dbContext.Products.Add(product);
        await _dbContext.SaveChangesAsync();
        return CreatedAtAction(nameof(GetById), new { id = product.Id }, product);
    }

    // Update a product - Bakers and Admins only
    [HttpPut("{id}")]
    [Authorize(Roles = "Baker,Admin")]
    public async Task<IActionResult> Update(int id, Product product)
    {
        if (id != product.Id)
        {
            return BadRequest();
        }

        _dbContext.Entry(product).State = EntityState.Modified;
        await _dbContext.SaveChangesAsync();
        return NoContent();
    }

    // Delete a product - Admins only
    [HttpDelete("{id}")]
    [Authorize(Roles = "Admin")]
    public async Task<IActionResult> Delete(int id)
    {
        var product = await _dbContext.Products.FindAsync(id);
        if (product == null)
        {
            return NotFound();
        }

        _dbContext.Products.Remove(product);
        await _dbContext.SaveChangesAsync();
        return NoContent();
    }

    // Update stock - Bakers and Admins only
    [HttpPatch("{id}/stock")]
    [Authorize(Roles = "Baker,Admin")]
    public async Task<IActionResult> UpdateStock(int id, [FromBody] int newStock)
    {
        var product = await _dbContext.Products.FindAsync(id);
        if (product == null)
        {
            return NotFound();
        }

        product.Stock = newStock;
        await _dbContext.SaveChangesAsync();
        return NoContent();
    }
}
```

Create a migration for the new Product model:

```bash
dotnet ef migrations add AddProducts
dotnet ef database update
```

## Understanding Role-Based Authorization

Let's break down how role-based authorization works in our application:

1. **Role Assignment**:
   - Users are assigned to roles (Admin, Baker, or Customer)
   - Roles are stored in the database

2. **Authorization Attributes**:
   - We use the `[Authorize]` attribute to restrict access to endpoints
   - We can specify which roles can access an endpoint with `[Authorize(Roles = "Role1,Role2")]`
   - Multiple roles can be specified, separated by commas

3. **Authorization Levels**:
   - Public endpoints: No `[Authorize]` attribute
   - User-only endpoints: `[Authorize]` with no roles specified
   - Role-specific endpoints: `[Authorize(Roles = "RoleName")]`

4. **Role Hierarchy**:
   - In our example, Admins have the most permissions
   - Bakers have more permissions than Customers
   - Customers have basic access

## Testing Role-Based Authorization

You can test the role-based authorization using Postman or Swagger:

1. Register a new user (they'll be a Customer by default)
2. Try to access the different endpoints:
   - GET `/api/product` should work for everyone
   - POST `/api/product` should only work for Bakers and Admins
   - DELETE `/api/product/{id}` should only work for Admins

3. As an Admin, promote a user to Baker:
   - POST `/api/userprofile/promote/{id}`

4. Log in as the promoted user and try the endpoints again

## Summary

In this chapter, we've added role-based authorization to our TinyTreats application. We've:

- Created roles (Admin, Baker, Customer)
- Assigned users to roles
- Added role-based authorization to our controllers
- Created methods to promote and demote users between roles

This gives us fine-grained control over what different users can do in our application. In the next chapter, we'll explore more advanced authorization techniques and secure our API endpoints.

[Next: Securing API Endpoints](./auth-securing-endpoints.md)