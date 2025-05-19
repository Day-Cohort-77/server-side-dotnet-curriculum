# Authentication Endpoints

In this chapter, we'll implement the authentication endpoints for our TinyTreats application. We'll create endpoints for user registration, login, and logout, following our organized approach of placing related endpoints in separate files.

## Understanding Authentication in ASP.NET Core

**ASP.NET Core Identity** provides a robust framework for handling authentication. It includes:

- User management (registration, login, logout)
- Password hashing and validation
- Cookie-based authentication
- Role-based authorization

We'll leverage these features to implement secure authentication for our TinyTreats application.

## Creating the Authentication Endpoints

Let's create a new file called `Endpoints/AuthEndpoints.cs` to contain all our authentication-related endpoints. The endpoints will allow users to register new accounts, authenticate with their credentials, retrieve their profile information, and logout.

```csharp
// Endpoints/AuthEndpoints.cs
using Microsoft.AspNetCore.Identity;
using System.Security.Claims;
using TinyTreats.Data;
using TinyTreats.DTO;
using TinyTreats.Models;
using Microsoft.EntityFrameworkCore;

namespace TinyTreats.Endpoints;

public static class AuthEndpoints
{
    public static void MapAuthEndpoints(this WebApplication app)
    {
        // Registration endpoint
        app.MapPost("/auth/register", async (
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
                // Assign the Customer role by default
                await userManager.AddToRoleAsync(user, "Customer");

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
        app.MapPost("/auth/login", async (
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
        app.MapPost("/auth/logout", async (SignInManager<IdentityUser> signInManager) =>
        {
            await signInManager.SignOutAsync();
            return Results.Ok();
        });

        // Get current user info
        app.MapGet("/auth/me", async (
            ClaimsPrincipal user,
            UserManager<IdentityUser> userManager,
            TinyTreatsDbContext dbContext) =>
        {
            // Get the user ID from the claims
            var identityUserId = user.FindFirstValue(ClaimTypes.NameIdentifier);

            if (identityUserId == null)
            {
                return Results.Unauthorized();
            }

            // Find the user profile
            var profile = await dbContext.UserProfiles
                .FirstOrDefaultAsync(up => up.IdentityUserId == identityUserId);

            if (profile == null)
            {
                return Results.NotFound();
            }

            // Get the user's roles
            var identityUser = await userManager.FindByIdAsync(identityUserId);
            var roles = await userManager.GetRolesAsync(identityUser);

            // Return the user profile with roles
            return Results.Ok(new
            {
                profile.Id,
                profile.FirstName,
                profile.LastName,
                profile.Address,
                Email = user.FindFirstValue(ClaimTypes.Email),
                Roles = roles
            });
        }).RequireAuthorization(); // This endpoint requires authentication
    }
}
```

## Understanding the Authentication Endpoints

Let's break down each endpoint:

### Registration Endpoint

The registration endpoint (`/auth/register`) handles user registration:

```csharp
app.MapPost("/auth/register", async (
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
        // Assign the Customer role by default
        await userManager.AddToRoleAsync(user, "Customer");

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
```

This endpoint:
1. Creates a new `IdentityUser` with the provided email
2. Attempts to create the user with the provided password
3. If successful, assigns the "Customer" role by default
4. Creates a `UserProfile` for the new user
5. Signs the user in
6. Returns a success response

### Login Endpoint

The login endpoint (`/auth/login`) handles user login:

```csharp
app.MapPost("/auth/login", async (
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
```

This endpoint:
1. Finds the user by email
2. Verifies the password
3. If successful, signs the user in
4. Returns a success response

### Logout Endpoint

The logout endpoint (`/auth/logout`) handles user logout:

```csharp
app.MapPost("/auth/logout", async (SignInManager<IdentityUser> signInManager) =>
{
    await signInManager.SignOutAsync();
    return Results.Ok();
});
```

This endpoint simply signs the user out and returns a success response.

### Current User Endpoint

The current user endpoint (`/auth/me`) returns information about the currently authenticated user:

```csharp
app.MapGet("/auth/me", async (
    ClaimsPrincipal user,
    UserManager<IdentityUser> userManager,
    TinyTreatsDbContext dbContext) =>
{
    // Get the user ID from the claims
    var identityUserId = user.FindFirstValue(ClaimTypes.NameIdentifier);

    if (identityUserId == null)
    {
        return Results.Unauthorized();
    }

    // Find the user profile
    var profile = await dbContext.UserProfiles
        .FirstOrDefaultAsync(up => up.IdentityUserId == identityUserId);

    if (profile == null)
    {
        return Results.NotFound();
    }

    // Get the user's roles
    var identityUser = await userManager.FindByIdAsync(identityUserId);
    var roles = await userManager.GetRolesAsync(identityUser);

    // Return the user profile with roles
    return Results.Ok(new
    {
        profile.Id,
        profile.FirstName,
        profile.LastName,
        profile.Address,
        Email = user.FindFirstValue(ClaimTypes.Email),
        Roles = roles
    });
}).RequireAuthorization(); // This endpoint requires authentication
```

This endpoint:
1. Gets the user ID from the claims
2. Finds the user profile
3. Gets the user's roles
4. Returns the user profile with roles
5. Requires authentication (`.RequireAuthorization()`)

## Dependency Injection

Notice how we're using dependency injection to get the services we need:

```csharp
UserManager<IdentityUser> userManager,
SignInManager<IdentityUser> signInManager,
TinyTreatsDbContext dbContext
```

ASP.NET Core's dependency injection system automatically provides these services when our endpoints are called.

## Authentication Flow

Let's understand the authentication flow in our application:

1. **Registration**:
   - User provides email, password, and profile information
   - System creates an Identity user and a user profile
   - User is assigned the "Customer" role by default
   - User is automatically signed in

2. **Login**:
   - User provides email and password
   - System verifies the credentials
   - User is signed in if credentials are valid

3. **Authentication**:
   - When a user is signed in, a cookie is created
   - This cookie is sent with subsequent requests
   - The system uses the cookie to identify the user

4. **Authorization**:
   - The system checks the user's roles to determine what they can access
   - Endpoints can be secured using `.RequireAuthorization()`

## Summary

In this chapter, we've implemented the authentication endpoints for our TinyTreats application:

- Registration endpoint for creating new users
- Login endpoint for authenticating users
- Logout endpoint for signing users out
- Current user endpoint for getting information about the authenticated user

These endpoints provide a solid foundation for user authentication in our application. In the next chapter, we'll implement role management endpoints to control what different users can do.

[Next: Role Management Endpoints](./tinytreats-role-endpoints.md)