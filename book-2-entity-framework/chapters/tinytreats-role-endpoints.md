# Role Management Endpoints

In this chapter, we'll implement role management endpoints for our TinyTreats application. These endpoints will allow administrators to create roles, assign roles to users, and manage user permissions.

## Understanding Roles in ASP.NET Core Identity

Roles in ASP.NET Core Identity provide a way to group users and assign permissions to those groups. For example, in our TinyTreats application, we have three roles:

1. **Admin**: Can manage everything, including users, roles, products, and orders
2. **Baker**: Can view and update order statuses
3. **Customer**: Can place orders and view their own order history

ASP.NET Core Identity provides a `RoleManager<IdentityRole>` class for managing roles and a `UserManager<IdentityUser>` class for assigning roles to users.

## Creating the Role Endpoints

Let's create a new file called `Endpoints/RoleEndpoints.cs` to contain all our role-related endpoints:

```csharp
// Endpoints/RoleEndpoints.cs
using Microsoft.AspNetCore.Identity;
using TinyTreats.DTO;
using Microsoft.EntityFrameworkCore;
using AutoMapper;

namespace TinyTreats.Endpoints;

public static class RoleEndpoints
{
    public static void MapRoleEndpoints(this WebApplication app)
    {
        // Create a new role - Admin only
        app.MapPost("/roles", async (
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
                return Results.Created($"/roles/{roleDto.Name}", new { name = roleDto.Name });
            }

            return Results.BadRequest(result.Errors);
        }).RequireAuthorization(policy => policy.RequireRole("Admin"));

        // Get all roles - Admin only
        app.MapGet("/roles", async (RoleManager<IdentityRole> roleManager) =>
        {
            var roles = await roleManager.Roles.Select(r => r.Name).ToListAsync();
            return Results.Ok(roles);
        }).RequireAuthorization(policy => policy.RequireRole("Admin"));

        // Assign a role to a user - Admin only
        app.MapPost("/users/roles", async (
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
        app.MapGet("/users/{email}/roles", async (
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
        app.MapDelete("/users/{email}/roles/{roleName}", async (
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

## Understanding the Role Endpoints

Let's break down each endpoint:

### Create Role Endpoint

The create role endpoint (`/roles`) allows administrators to create new roles:

```csharp
app.MapPost("/roles", async (
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
        return Results.Created($"/roles/{roleDto.Name}", new { name = roleDto.Name });
    }

    return Results.BadRequest(result.Errors);
}).RequireAuthorization(policy => policy.RequireRole("Admin"));
```

This endpoint:
1. Checks if the role already exists
2. Creates the role if it doesn't exist
3. Returns a 201 Created response with the role name
4. Requires the "Admin" role (`.RequireAuthorization(policy => policy.RequireRole("Admin"))`)

### Get All Roles Endpoint

The get all roles endpoint (`/roles`) returns a list of all roles:

```csharp
app.MapGet("/roles", async (RoleManager<IdentityRole> roleManager) =>
{
    var roles = await roleManager.Roles.Select(r => r.Name).ToListAsync();
    return Results.Ok(roles);
}).RequireAuthorization(policy => policy.RequireRole("Admin"));
```

This endpoint:
1. Gets all roles from the role manager
2. Returns a 200 OK response with the role names
3. Requires the "Admin" role

### Assign Role Endpoint

The assign role endpoint (`/users/roles`) assigns a role to a user:

```csharp
app.MapPost("/users/roles", async (
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
```

This endpoint:
1. Finds the user by email
2. Checks if the role exists
3. Adds the user to the role
4. Returns a 204 No Content response if successful
5. Requires the "Admin" role

### Get User Roles Endpoint

The get user roles endpoint (`/users/{email}/roles`) returns the roles for a specific user:

```csharp
app.MapGet("/users/{email}/roles", async (
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
```

This endpoint:
1. Finds the user by email
2. Gets the user's roles
3. Returns a 200 OK response with the roles
4. Requires the "Admin" role

### Remove Role Endpoint

The remove role endpoint (`/users/{email}/roles/{roleName}`) removes a role from a user:

```csharp
app.MapDelete("/users/{email}/roles/{roleName}", async (
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
```

This endpoint:
1. Finds the user by email
2. Checks if the role exists
3. Checks if the user is in the role
4. Removes the user from the role
5. Returns a 204 No Content response if successful
6. Requires the "Admin" role

## Role-Based Authorization

Notice how we're using role-based authorization to secure our endpoints:

```csharp
.RequireAuthorization(policy => policy.RequireRole("Admin"))
```

This ensures that only users in the "Admin" role can access these endpoints. If a user without the "Admin" role tries to access these endpoints, they'll receive a 403 Forbidden response.

## Testing Role Management

You can test these endpoints using a tool like Yaak:

1. Login as an admin:
   ```http
   POST /auth/login
   Content-Type: application/json

   {
     "email": "admin@tinytreats.com",
     "password": "Admin123!"
   }
   ```

2. Create a new role:
   ```http
   POST /roles
   Content-Type: application/json

   {
     "name": "Manager"
   }
   ```

3. Get all roles:
   ```http
   GET /roles
   ```

4. Assign a role to a user:
   ```http
   POST /users/roles
   Content-Type: application/json

   {
     "email": "customer1@example.com",
     "roleName": "Baker"
   }
   ```

5. Get roles for a user:
   ```http
   GET /users/customer1@example.com/roles
   ```

6. Remove a role from a user:
   ```http
   DELETE /users/customer1@example.com/roles/Baker
   ```

## Summary

In this chapter, we've implemented role management endpoints for our TinyTreats application:

- Create role endpoint for creating new roles
- Get all roles endpoint for listing all roles
- Assign role endpoint for assigning roles to users
- Get user roles endpoint for listing a user's roles
- Remove role endpoint for removing roles from users

These endpoints provide a way for administrators to manage user permissions in our application. In the next chapter, we'll implement product management endpoints to allow users to browse and manage bakery products.

[Next: Authenticating with the client](./tinytreats-role-client-login.md)