# User Roles and Authorization

In this chapter, we'll add role-based authorization to our TinyTreats application. We'll create different user roles and control what each role can do in the application, all while maintaining our organized endpoint structure.

## Understanding Roles in ASP.NET Core Identity

Roles in ASP.NET Core Identity are a way to group users and assign permissions to those groups. For example, you might have roles like "Admin", "Baker", and "Customer", each with different permissions.

ASP.NET Core Identity provides a `RoleManager<IdentityRole>` class for managing roles and a `UserManager<IdentityUser>` class for assigning roles to users.

## Setting Up Roles

Let's start by adding role management to our application. We've already added the necessary services in `Program.cs`:

```csharp
builder.Services.AddIdentityCore<IdentityUser>(options => {
    // Password requirements...
})
.AddRoles<IdentityRole>() // Add role management
.AddEntityFrameworkStores<TinyTreatsDbContext>()
.AddSignInManager();
```

Now, let's create a new endpoint class for role management. Create a new file called `Endpoints/RoleEndpoints.cs`:

```csharp
// Endpoints/RoleEndpoints.cs
using Microsoft.AspNetCore.Identity;
using System.Security.Claims;
using TinyTreats.Data;

namespace TinyTreats.Endpoints;

public static class RoleEndpoints
{
    // DTO for role creation
    public class RoleDto
    {
        public string Name { get; set; }
    }

    // DTO for assigning a role to a user
    public class UserRoleDto
    {
        public string Email { get; set; }
        public string RoleName { get; set; }
    }

    public static void MapRoleEndpoints(this WebApplication app)
    {
        // Create a new role - Admin only
        app.MapPost("/api/roles", async (
            RoleDto roleDto,
            RoleManager<IdentityRole> roleManager) =>
        {
            // Check if the role already exists
            if (await roleManager.RoleExistsAsync(roleDto.Name))
            {
                return Results.BadRequest($"Role '{roleDto.Name}' already exists.");
            }

            // Create the role
            var result = await roleManager.CreateAsync(new IdentityRole(roleDto.Name));

            if (result.Succeeded)
            {
                return Results.Created($"/api/roles/{roleDto.Name}", new { name = roleDto.Name });
            }

            return Results.BadRequest(result.Errors);
        }).RequireAuthorization(policy => policy.RequireRole("Admin"));

        // Get all roles - Admin only
        app.MapGet("/api/roles", async (RoleManager<IdentityRole> roleManager) =>
        {
            var roles = roleManager.Roles.Select(r => r.Name).ToList();
            return Results.Ok(roles);
        }).RequireAuthorization(policy => policy.RequireRole("Admin"));

        // Assign a role to a user - Admin only
        app.MapPost("/api/users/roles", async (
            UserRoleDto userRoleDto,
            UserManager<IdentityUser> userManager,
            RoleManager<IdentityRole> roleManager) =>
        {
            // Find the user
            var user = await userManager.FindByEmailAsync(userRoleDto.Email);
            if (user == null)
            {
                return Results.NotFound($"User with email '{userRoleDto.Email}' not found.");
            }

            // Check if the role exists
            if (!await roleManager.RoleExistsAsync(userRoleDto.RoleName))
            {
                return Results.NotFound($"Role '{userRoleDto.RoleName}' not found.");
            }

            // Add the user to the role
            var result = await userManager.AddToRoleAsync(user, userRoleDto.RoleName);

            if (result.Succeeded)
            {
                return Results.NoContent();
            }

            return Results.BadRequest(result.Errors);
        }).RequireAuthorization(policy => policy.RequireRole("Admin"));

        // Get roles for a user - Admin only
        app.MapGet("/api/users/{email}/roles", async (
            string email,
            UserManager<IdentityUser> userManager) =>
        {
            // Find the user
            var user = await userManager.FindByEmailAsync(email);
            if (user == null)
            {
                return Results.NotFound($"User with email '{email}' not found.");
            }

            // Get the user's roles
            var roles = await userManager.GetRolesAsync(user);

            return Results.Ok(roles);
        }).RequireAuthorization(policy => policy.RequireRole("Admin"));

        // Remove a role from a user - Admin only
        app.MapDelete("/api/users/{email}/roles/{roleName}", async (
            string email,
            string roleName,
            UserManager<IdentityUser> userManager,
            RoleManager<IdentityRole> roleManager) =>
        {
            // Find the user
            var user = await userManager.FindByEmailAsync(email);
            if (user == null)
            {
                return Results.NotFound($"User with email '{email}' not found.");
            }

            // Check if the role exists
            if (!await roleManager.RoleExistsAsync(roleName))
            {
                return Results.NotFound($"Role '{roleName}' not found.");
            }

            // Check if the user is in the role
            if (!await userManager.IsInRoleAsync(user, roleName))
            {
                return Results.BadRequest($"User '{email}' is not in role '{roleName}'.");
            }

            // Remove the user from the role
            var result = await userManager.RemoveFromRoleAsync(user, roleName);

            if (result.Succeeded)
            {
                return Results.NoContent();
            }

            return Results.BadRequest(result.Errors);
        }).RequireAuthorization(policy => policy.RequireRole("Admin"));
    }
}
```

## Updating Program.cs

Now, let's update `Program.cs` to use our new role endpoints:

```csharp
// Program.cs
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

// Configure authorization policies
builder.Services.AddAuthorization(options =>
{
    // Add a policy for bakery staff (Admin or Baker)
    options.AddPolicy("BakeryStaff", policy =>
        policy.RequireRole("Admin", "Baker"));
});

builder.Services.AddEndpointsApiExplorer();

var app = builder.Build();

// Ensure that HTTPS protocol is used
app.UseHttpsRedirection();

// Add authentication middleware
app.UseAuthentication();
app.UseAuthorization();

// Map API endpoints
app.MapAuthEndpoints();
app.MapRoleEndpoints();
app.MapOrderEndpoints();

app.Run();
```

## Seeding Roles and Admin User

Let's update our database context to seed roles and assign the admin user to the Admin role:

```csharp
// Data/TinyTreatsDbContext.cs
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

    // Seed an admin user
    modelBuilder.Entity<IdentityUser>().HasData(new IdentityUser
    {
        Id = "dbc40bc6-0829-4ac5-a3ed-180f5e916a5f",
        UserName = "admin@tinytreats.com",
        Email = "admin@tinytreats.com",
        NormalizedEmail = "ADMIN@TINYTREATS.COM",
        NormalizedUserName = "ADMIN@TINYTREATS.COM",
        EmailConfirmed = true,
        // This hashes the password "Admin123!"
        PasswordHash = new PasswordHasher<IdentityUser>().HashPassword(null, "Admin123!")
    });

    // Seed a user profile for the admin
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
        RoleId = "fab4fac1-c546-41de-aebc-a14da6895711",
        UserId = "dbc40bc6-0829-4ac5-a3ed-180f5e916a5f"
    });
}
```

## Creating a Migration for Roles

Now, let's create a migration to add roles to the database:

```bash
dotnet ef migrations add AddRoles
dotnet ef database update
```

## Testing Role-Based Authorization

Let's test our role-based authorization:

1. Login as the admin user:
   - POST to `/api/auth/login`
   - Body:
     ```json
     {
       "email": "admin@tinytreats.com",
       "password": "Admin123!"
     }
     ```

2. Create a new role:
   - POST to `/api/roles`
   - Body:
     ```json
     {
       "name": "Manager"
     }
     ```

3. Register a new user:
   - POST to `/api/auth/register`
   - Body:
     ```json
     {
       "email": "baker@example.com",
       "password": "Baker123!",
       "firstName": "Jane",
       "lastName": "Baker",
       "address": "789 Bakery St"
     }
     ```

4. Assign the Baker role to the new user:
   - POST to `/api/users/roles`
   - Body:
     ```json
     {
       "email": "baker@example.com",
       "roleName": "Baker"
     }
     ```

5. Get the roles for the new user:
   - GET to `/api/users/baker@example.com/roles`

## Understanding Role-Based Authorization

Let's break down how role-based authorization works in our application:

1. **Role Definition**:
   - Roles are defined in the database using the `IdentityRole` entity
   - We've seeded three roles: Admin, Baker, and Customer

2. **Role Assignment**:
   - Users are assigned to roles using the `UserManager.AddToRoleAsync` method
   - We've seeded the admin user with the Admin role

3. **Role-Based Access Control**:
   - Endpoints are secured using the `.RequireAuthorization(policy => policy.RequireRole("Admin"))` method
   - This restricts access to users in the specified role

4. **Policy-Based Access Control**:
   - We've defined a "BakeryStaff" policy that includes both Admin and Baker roles
   - Endpoints can be secured using `.RequireAuthorization("BakeryStaff")`

## Benefits of Organized Endpoints

By organizing our role management endpoints in a separate file, we gain several benefits:

1. **Better organization** - Role management endpoints are grouped together, making them easier to find and modify.
2. **Improved maintainability** - The `Program.cs` file remains clean and focused on configuration.
3. **Scalability** - As your application grows, you can add new endpoint classes without cluttering `Program.cs`.
4. **Testability** - Endpoint classes can be tested independently.

## Summary

In this chapter, we've added role-based authorization to our TinyTreats application:

- We've created a `RoleEndpoints.cs` file to manage roles
- We've seeded roles and assigned the admin user to the Admin role
- We've secured endpoints based on roles
- We've defined authorization policies for more complex access control

With these features, we can control what different users can do in our application based on their roles. This is a powerful way to implement security in your application.

In the next chapter, we'll explore more advanced techniques for securing API endpoints.

[Next: Securing API Endpoints](./auth-securing-endpoints.md)