# User Registration and Login

In this chapter, we'll set up a simple project with ASP.NET Core Identity and implement user registration and login functionality using Minimal API. We'll follow our organized approach by placing authentication endpoints in a separate file and DTOs in their own directory.

## Project Setup

Let's create a new project called "Tiny Treats" - a simple bakery management system where we'll implement authentication.

1. Create a new ASP.NET Core Minimal API project:
   ```bash
   dotnet new web -n TinyTreats
   cd TinyTreats
   ```

2. Add the required NuGet packages:
   ```bash
   dotnet add package Microsoft.EntityFrameworkCore.Design --version 8.0.0
   dotnet add package Microsoft.EntityFrameworkCore.Tools --version 8.0.0
   dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL --version 8.0.0
   dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore --version 8.0.0
   ```

3. Set up user secrets for database connection:
   ```bash
   dotnet user-secrets init
   dotnet user-secrets set "TinyTreatsDbConnectionString" "Host=localhost;Port=5432;Username=postgres;Password=<your-password>;Database=TinyTreats"
   ```

4. Create the necessary folders for our organized project structure:
   ```bash
   mkdir Models Data Endpoints DTOs
   ```

## Creating the Data Models

First, let's create our basic data models. We'll start with a simple `UserProfile` model that extends the Identity user information:

```csharp
// Models/UserProfile.cs
using Microsoft.AspNetCore.Identity;

namespace TinyTreats.Models;

public class UserProfile
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Address { get; set; }

    // This connects to the ASP.NET Core Identity user
    public string IdentityUserId { get; set; }
    public IdentityUser IdentityUser { get; set; }
}
```

## Setting Up the Database Context

Now, let's create our database context that inherits from `IdentityDbContext`:

```csharp
// Data/TinyTreatsDbContext.cs
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Identity.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore;
using TinyTreats.Models;

namespace TinyTreats.Data;

public class TinyTreatsDbContext : IdentityDbContext<IdentityUser>
{
    public DbSet<UserProfile> UserProfiles { get; set; }

    public TinyTreatsDbContext(DbContextOptions<TinyTreatsDbContext> options) : base(options) { }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

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
    }
}
```

## Creating DTOs for Authentication

Following best practices, we'll create a separate file for our DTOs (Data Transfer Objects). These objects will be used to transfer data between our API and clients:

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

## Creating the Authentication Endpoints

Following our organized approach, we'll create a separate file for our authentication endpoints. Create a new file called `Endpoints/AuthEndpoints.cs`:

```csharp
// Endpoints/AuthEndpoints.cs
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

## Configuring the Application with Minimal API

Update the `Program.cs` file to configure Identity and our database using the Minimal API approach:

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

builder.Services.AddAuthorization();
builder.Services.AddEndpointsApiExplorer();

var app = builder.Build();

// Ensure that HTTPS protocol is used
app.UseHttpsRedirection();

// Add authentication middleware
app.UseAuthentication();
app.UseAuthorization();

// Map API endpoints
app.MapAuthEndpoints();

app.Run();
```

## Creating the Database

Now let's create the database using Entity Framework migrations:

```bash
dotnet ef migrations add InitialCreate
dotnet ef database update
```

## Testing the Authentication

You can test the authentication endpoints using a tool like Yaak:

1. Register a new user:
   - POST to `/api/auth/register`
   - Body:
     ```json
     {
       "email": "user@example.com",
       "password": "Password123!",
       "firstName": "John",
       "lastName": "Doe",
       "address": "456 Main St"
     }
     ```

2. Login with the user:
   - POST to `/api/auth/login`
   - Body:
     ```json
     {
       "email": "user@example.com",
       "password": "Password123!"
     }
     ```

3. Get the current user's info:
   - GET to `/api/auth/me`
   - The cookie should be automatically included by your browser

4. Logout:
   - POST to `/api/auth/logout`

## Understanding What's Happening

Let's break down what's happening in our authentication system:

1. **Registration**:
   - We create a new `IdentityUser` with the email and password
   - We create a `UserProfile` linked to that user
   - We sign the user in, which creates a cookie

2. **Login**:
   - We find the user by email
   - We verify the password
   - We sign the user in, which creates a cookie

3. **Me Endpoint**:
   - We get the user ID from the claims in the cookie
   - We find the user profile associated with that ID
   - We return the user's information

4. **Logout**:
   - We sign the user out, which removes the cookie

## Benefits of Organized Code Structure

By organizing our code with separate directories for DTOs and endpoints, we gain several benefits:

1. **Better organization** - Authentication endpoints and DTOs are grouped logically, making them easier to find and modify.
2. **Improved maintainability** - The `Program.cs` file remains clean and focused on configuration.
3. **Scalability** - As your application grows, you can add new endpoint classes and DTOs without cluttering existing files.
4. **Testability** - Endpoint classes and DTOs can be tested independently.
5. **Reusability** - DTOs can be reused across different endpoints and services.
6. **Single Responsibility Principle** - Each class has a single responsibility, making the code easier to understand and maintain.

## Summary

In this chapter, we've set up a basic authentication system using ASP.NET Core Identity with Minimal API. We've implemented:

- User registration
- User login
- User logout
- Getting the current user's information

We've also organized our code by:
- Placing endpoints in a separate file
- Moving DTOs to their own directory
- Following the Single Responsibility Principle

This provides a clean, maintainable, and scalable architecture for our application.

This provides the foundation for our authentication system. In the next chapter, we'll add role-based authorization to control what different users can do in our application.

[Next: User Roles and Authorization](./auth-roles-authorization.md)