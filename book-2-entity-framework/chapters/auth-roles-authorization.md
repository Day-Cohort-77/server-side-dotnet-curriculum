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
using TinyTreats.DTO;

namespace TinyTreats.Endpoints;

public static class RoleEndpoints
{
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
using Microsoft.AspNetCore.Authentication.Cookies;
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
        options.Cookie.HttpOnly = true; // Prevent JavaScript access to the cookie
        options.ExpireTimeSpan = TimeSpan.FromHours(8);
        options.Cookie.SameSite = SameSiteMode.Lax;
        options.Cookie.SecurePolicy = CookieSecurePolicy.SameAsRequest; // Allow both HTTP and HTTPS in development
        options.Events = new CookieAuthenticationEvents
        {
            OnRedirectToLogin = context =>
            {
                // Instead of redirecting to login page, return 401 Unauthorized for API requests
                context.Response.StatusCode = StatusCodes.Status401Unauthorized;
                return Task.CompletedTask;
            }
        };
    });

builder.Services.AddAuthorization();

// Configure CORS
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowLocalhost3000", policy =>
    {
        policy.WithOrigins("http://localhost:3000", "http://localhost:5173") // Added Vite's default port
              .AllowAnyHeader()
              .AllowAnyMethod()
              .AllowCredentials(); // Allow credentials (cookies)
    });
});

builder.Services.AddEndpointsApiExplorer();

var app = builder.Build();

// Ensure that HTTPS protocol is used
app.UseHttpsRedirection();

// Use CORS middleware
app.UseCors("AllowLocalhost3000");

// Add authentication middleware
app.UseAuthentication();
app.UseAuthorization();

// Map API endpoints
app.MapAuthEndpoints();

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

    // Seed users
    modelBuilder.Entity<IdentityUser>().HasData(
        // Admin user
        new IdentityUser
        {
            Id = "dbc40bc6-0829-4ac5-a3ed-180f5e916a5f",
            UserName = "admin@tinytreats.com",
            Email = "admin@tinytreats.com",
            NormalizedEmail = "ADMIN@TINYTREATS.COM",
            NormalizedUserName = "ADMIN@TINYTREATS.COM",
            EmailConfirmed = true,
            // This hashes the password "Admin123!"
            PasswordHash = new PasswordHasher<IdentityUser>().HashPassword(null, "Admin123!")
        },

        // Baker users
        new IdentityUser
        {
            Id = "e2cfe4e6-5437-4efb-9a66-8d1371796bda",
            UserName = "baker1@tinytreats.com",
            Email = "baker1@tinytreats.com",
            NormalizedEmail = "BAKER1@TINYTREATS.COM",
            NormalizedUserName = "BAKER1@TINYTREATS.COM",
            EmailConfirmed = true,
            PasswordHash = new PasswordHasher<IdentityUser>().HashPassword(null, "Baker123!")
        },
        new IdentityUser
        {
            Id = "a1ffd800-9189-4a69-a24a-9b8c094f12a5",
            UserName = "baker2@tinytreats.com",
            Email = "baker2@tinytreats.com",
            NormalizedEmail = "BAKER2@TINYTREATS.COM",
            NormalizedUserName = "BAKER2@TINYTREATS.COM",
            EmailConfirmed = true,
            PasswordHash = new PasswordHasher<IdentityUser>().HashPassword(null, "Baker123!")
        },

        // Customer users
        new IdentityUser
        {
            Id = "b9c6f5e4-d4d5-4a16-9551-b7e5e859c35a",
            UserName = "customer1@example.com",
            Email = "customer1@example.com",
            NormalizedEmail = "CUSTOMER1@EXAMPLE.COM",
            NormalizedUserName = "CUSTOMER1@EXAMPLE.COM",
            EmailConfirmed = true,
            PasswordHash = new PasswordHasher<IdentityUser>().HashPassword(null, "Customer123!")
        },
        new IdentityUser
        {
            Id = "c9d6f5e4-d4d5-4a16-9551-b7e5e859c35b",
            UserName = "customer2@example.com",
            Email = "customer2@example.com",
            NormalizedEmail = "CUSTOMER2@EXAMPLE.COM",
            NormalizedUserName = "CUSTOMER2@EXAMPLE.COM",
            EmailConfirmed = true,
            PasswordHash = new PasswordHasher<IdentityUser>().HashPassword(null, "Customer123!")
        },
        new IdentityUser
        {
            Id = "d9d6f5e4-d4d5-4a16-9551-b7e5e859c35c",
            UserName = "customer3@example.com",
            Email = "customer3@example.com",
            NormalizedEmail = "CUSTOMER3@EXAMPLE.COM",
            NormalizedUserName = "CUSTOMER3@EXAMPLE.COM",
            EmailConfirmed = true,
            PasswordHash = new PasswordHasher<IdentityUser>().HashPassword(null, "Customer123!")
        },
        new IdentityUser
        {
            Id = "e9d6f5e4-d4d5-4a16-9551-b7e5e859c35d",
            UserName = "customer4@example.com",
            Email = "customer4@example.com",
            NormalizedEmail = "CUSTOMER4@EXAMPLE.COM",
            NormalizedUserName = "CUSTOMER4@EXAMPLE.COM",
            EmailConfirmed = true,
            PasswordHash = new PasswordHasher<IdentityUser>().HashPassword(null, "Customer123!")
        },
        new IdentityUser
        {
            Id = "f9d6f5e4-d4d5-4a16-9551-b7e5e859c35e",
            UserName = "customer5@example.com",
            Email = "customer5@example.com",
            NormalizedEmail = "CUSTOMER5@EXAMPLE.COM",
            NormalizedUserName = "CUSTOMER5@EXAMPLE.COM",
            EmailConfirmed = true,
            PasswordHash = new PasswordHasher<IdentityUser>().HashPassword(null, "Customer123!")
        }
    );

    // Seed user profiles
    modelBuilder.Entity<UserProfile>().HasData(
        // Admin profile
        new UserProfile
        {
            Id = 1,
            IdentityUserId = "dbc40bc6-0829-4ac5-a3ed-180f5e916a5f",
            FirstName = "Admin",
            LastName = "User",
            Address = "123 Bakery Lane"
        },

        // Baker profiles
        new UserProfile
        {
            Id = 2,
            IdentityUserId = "e2cfe4e6-5437-4efb-9a66-8d1371796bda",
            FirstName = "Jane",
            LastName = "Baker",
            Address = "456 Pastry Ave"
        },
        new UserProfile
        {
            Id = 3,
            IdentityUserId = "a1ffd800-9189-4a69-a24a-9b8c094f12a5",
            FirstName = "John",
            LastName = "Dough",
            Address = "789 Cupcake Blvd"
        },

        // Customer profiles
        new UserProfile
        {
            Id = 4,
            IdentityUserId = "b9c6f5e4-d4d5-4a16-9551-b7e5e859c35a",
            FirstName = "Alice",
            LastName = "Johnson",
            Address = "101 Sweet St"
        },
        new UserProfile
        {
            Id = 5,
            IdentityUserId = "c9d6f5e4-d4d5-4a16-9551-b7e5e859c35b",
            FirstName = "Bob",
            LastName = "Smith",
            Address = "202 Dessert Dr"
        },
        new UserProfile
        {
            Id = 6,
            IdentityUserId = "d9d6f5e4-d4d5-4a16-9551-b7e5e859c35c",
            FirstName = "Carol",
            LastName = "Williams",
            Address = "303 Frosting Ln"
        },
        new UserProfile
        {
            Id = 7,
            IdentityUserId = "e9d6f5e4-d4d5-4a16-9551-b7e5e859c35d",
            FirstName = "David",
            LastName = "Brown",
            Address = "404 Cookie Ct"
        },
        new UserProfile
        {
            Id = 8,
            IdentityUserId = "f9d6f5e4-d4d5-4a16-9551-b7e5e859c35e",
            FirstName = "Emma",
            LastName = "Davis",
            Address = "505 Muffin Ave"
        }
    );

    // Assign users to roles
    modelBuilder.Entity<IdentityUserRole<string>>().HasData(
        // Admin role assignment
        new IdentityUserRole<string>
        {
            RoleId = "fab4fac1-c546-41de-aebc-a14da6895711",
            UserId = "dbc40bc6-0829-4ac5-a3ed-180f5e916a5f"
        },

        // Baker role assignments
        new IdentityUserRole<string>
        {
            RoleId = "c7b013f0-5201-4317-abd8-c211f91b7330",
            UserId = "e2cfe4e6-5437-4efb-9a66-8d1371796bda"
        },
        new IdentityUserRole<string>
        {
            RoleId = "c7b013f0-5201-4317-abd8-c211f91b7330",
            UserId = "a1ffd800-9189-4a69-a24a-9b8c094f12a5"
        },

        // Customer role assignments
        new IdentityUserRole<string>
        {
            RoleId = "2c5e174e-3b0e-446f-86af-483d56fd7210",
            UserId = "b9c6f5e4-d4d5-4a16-9551-b7e5e859c35a"
        },
        new IdentityUserRole<string>
        {
            RoleId = "2c5e174e-3b0e-446f-86af-483d56fd7210",
            UserId = "c9d6f5e4-d4d5-4a16-9551-b7e5e859c35b"
        },
        new IdentityUserRole<string>
        {
            RoleId = "2c5e174e-3b0e-446f-86af-483d56fd7210",
            UserId = "d9d6f5e4-d4d5-4a16-9551-b7e5e859c35c"
        },
        new IdentityUserRole<string>
        {
            RoleId = "2c5e174e-3b0e-446f-86af-483d56fd7210",
            UserId = "e9d6f5e4-d4d5-4a16-9551-b7e5e859c35d"
        },
        new IdentityUserRole<string>
        {
            RoleId = "2c5e174e-3b0e-446f-86af-483d56fd7210",
            UserId = "f9d6f5e4-d4d5-4a16-9551-b7e5e859c35e"
        }
    );
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