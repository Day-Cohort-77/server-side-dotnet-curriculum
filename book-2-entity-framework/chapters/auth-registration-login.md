# User Registration and Login

In this chapter, we'll set up a simple project with ASP.NET Core Identity and implement user registration and login functionality.

## Project Setup

Let's create a new project called "TinyTreats" - a simple bakery management system where we'll implement authentication.

1. Create a new ASP.NET Core Web API project:
   ```bash
   dotnet new webapi -n TinyTreats
   cd TinyTreats
   ```

2. Add the required NuGet packages:
   ```bash
   dotnet add package Microsoft.EntityFrameworkCore.Design
   dotnet add package Microsoft.EntityFrameworkCore.Tools
   dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
   dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
   ```

3. Set up user secrets for database connection:
   ```bash
   dotnet user-secrets init
   dotnet user-secrets set "TinyTreatsDbConnectionString" "Host=localhost;Port=5432;Username=postgres;Password=<your-password>;Database=TinyTreats"
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

## Configuring the Application

Update the `Program.cs` file to configure Identity and our database:

```csharp
// Program.cs
using Microsoft.AspNetCore.Identity;
using Microsoft.EntityFrameworkCore;
using TinyTreats.Data;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddControllers();

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
.AddEntityFrameworkStores<TinyTreatsDbContext>(); // Use our DbContext

// Configure authentication with cookies
builder.Services.AddAuthentication("Identity.Application")
    .AddCookie("Identity.Application", options =>
    {
        options.Cookie.Name = "TinyTreatsAuth";
        options.ExpireTimeSpan = TimeSpan.FromHours(8);
    });

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

app.UseHttpsRedirection();

// Add authentication middleware
app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();

app.Run();
```

## Creating the Auth Controller

Now, let's create a controller to handle user registration and login:

```csharp
// Controllers/AuthController.cs
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Mvc;
using System.Security.Claims;
using TinyTreats.Data;
using TinyTreats.Models;

namespace TinyTreats.Controllers;

[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    private TinyTreatsDbContext _dbContext;
    private UserManager<IdentityUser> _userManager;
    private SignInManager<IdentityUser> _signInManager;

    public AuthController(
        TinyTreatsDbContext context,
        UserManager<IdentityUser> userManager,
        SignInManager<IdentityUser> signInManager)
    {
        _dbContext = context;
        _userManager = userManager;
        _signInManager = signInManager;
    }

    // Registration endpoint
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

    // Login endpoint
    [HttpPost("login")]
    public async Task<IActionResult> Login([FromBody] LoginDto login)
    {
        // Find the user by email
        var user = await _userManager.FindByEmailAsync(login.Email);

        if (user == null)
        {
            return Unauthorized();
        }

        // Verify the password
        var result = await _signInManager.CheckPasswordSignInAsync(user, login.Password, false);

        if (result.Succeeded)
        {
            // Sign in the user
            await _signInManager.SignInAsync(user, isPersistent: false);
            return Ok();
        }

        return Unauthorized();
    }

    // Logout endpoint
    [HttpPost("logout")]
    public async Task<IActionResult> Logout()
    {
        await _signInManager.SignOutAsync();
        return Ok();
    }

    // Get current user info
    [HttpGet("me")]
    public IActionResult Me()
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

        // Return the user profile
        return Ok(new
        {
            profile.Id,
            profile.FirstName,
            profile.LastName,
            profile.Address,
            Email = User.FindFirstValue(ClaimTypes.Email)
        });
    }
}

// DTOs for registration and login
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

## Creating the Database

Now let's create the database using Entity Framework migrations:

```bash
dotnet ef migrations add InitialCreate
dotnet ef database update
```

## Testing the Authentication

You can test the authentication endpoints using a tool like Postman or Swagger:

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

## Summary

In this chapter, we've set up a basic authentication system using ASP.NET Core Identity. We've implemented:

- User registration
- User login
- User logout
- Getting the current user's information

This provides the foundation for our authentication system. In the next chapter, we'll add role-based authorization to control what different users can do in our application.

[Next: User Roles and Authorization](./auth-roles-authorization.md)