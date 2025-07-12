# Program.cs and Database Migrations

In this final chapter, we'll implement the `Program.cs` file to tie everything together and provide instructions for creating migrations and updating the database. This is where we'll configure all the services and middleware for our TinyTreats application.

## Understanding Program.cs in Minimal API

In ASP.NET Core Minimal API, the `Program.cs` file is the entry point of the application. It's responsible for:

1. **Configuring services**: Adding services to the dependency injection container
2. **Configuring middleware**: Setting up the HTTP request pipeline
3. **Mapping endpoints**: Registering API endpoints
4. **Starting the application**: Running the web server

Let's create a comprehensive `Program.cs` file that brings together all the components we've built so far.

## Creating the Program.cs File

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
    // Password requirements
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
            },
            OnRedirectToAccessDenied = context =>
            {
                // Instead of redirecting to access denied page, return 403 Forbidden for API requests
                context.Response.StatusCode = StatusCodes.Status403Forbidden;
                return Task.CompletedTask;
            }
        };
    });

// Configure authorization policies
builder.Services.AddAuthorization(options =>
{
    // Add a policy for bakery staff (Admin or Baker)
    options.AddPolicy("BakeryStaff", policy =>
        policy.RequireRole("Admin", "Baker"));
});

// Configure CORS
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowLocalhost", policy =>
    {
        policy.WithOrigins("http://localhost:3000", "http://localhost:5173") // React and Vite default ports
              .AllowAnyHeader()
              .AllowAnyMethod()
              .AllowCredentials(); // Allow credentials (cookies)
    });
});

// Add services for Swagger
builder.Services.AddEndpointsApiExplorer();

var app = builder.Build();

// Configure the HTTP request pipeline
if (app.Environment.IsDevelopment())
{
    // Development-specific middleware
    app.UseDeveloperExceptionPage();
}

// Ensure that HTTPS protocol is used
app.UseHttpsRedirection();

// Use CORS middleware
app.UseCors("AllowLocalhost");

// Add authentication middleware
app.UseAuthentication();
app.UseAuthorization();

// Map API endpoints
app.MapAuthEndpoints();
app.MapRoleEndpoints();
app.MapProductEndpoints();
app.MapOrderEndpoints();

// Add a simple health check endpoint
app.MapGet("/", () => "TinyTreats API is running!");

app.Run();
```

## Understanding the Program.cs Implementation

Let's break down the key components of our `Program.cs` file:

### Database Configuration

```csharp
// Configure database
builder.Services.AddDbContext<TinyTreatsDbContext>(options =>
    options.UseNpgsql(builder.Configuration["TinyTreatsDbConnectionString"]));
```

This configures Entity Framework Core to use our `TinyTreatsDbContext` with PostgreSQL. The connection string is retrieved from the configuration (user secrets in development).

### Identity Configuration

```csharp
// Configure Identity
builder.Services.AddIdentityCore<IdentityUser>(options =>
{
    // Password requirements
    options.Password.RequireDigit = true;
    options.Password.RequiredLength = 6;
    options.Password.RequireLowercase = true;
    options.Password.RequireNonAlphanumeric = false;
    options.Password.RequireUppercase = true;
})
.AddRoles<IdentityRole>() // Add role management
.AddEntityFrameworkStores<TinyTreatsDbContext>() // Use our DbContext
.AddSignInManager(); // Add SignInManager
```

This configures ASP.NET Core Identity with:
- Password requirements
- Role management
- Entity Framework Core storage
- Sign-in management

### Authentication Configuration

```csharp
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
            },
            OnRedirectToAccessDenied = context =>
            {
                // Instead of redirecting to access denied page, return 403 Forbidden for API requests
                context.Response.StatusCode = StatusCodes.Status403Forbidden;
                return Task.CompletedTask;
            }
        };
    });
```

This configures cookie-based authentication with:
- Custom cookie name and settings
- 8-hour expiration
- HTTP-only cookies for security
- Custom redirect handling for API requests

### Authorization Configuration

```csharp
// Configure authorization policies
builder.Services.AddAuthorization(options =>
{
    // Add a policy for bakery staff (Admin or Baker)
    options.AddPolicy("BakeryStaff", policy =>
        policy.RequireRole("Admin", "Baker"));
});
```

This configures authorization policies, including a "BakeryStaff" policy that requires either the "Admin" or "Baker" role.

### CORS Configuration

```csharp
// Configure CORS
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowLocalhost", policy =>
    {
        policy.WithOrigins("http://localhost:3000", "http://localhost:5173") // React and Vite default ports
              .AllowAnyHeader()
              .AllowAnyMethod()
              .AllowCredentials(); // Allow credentials (cookies)
    });
});
```

This configures Cross-Origin Resource Sharing (CORS) to allow requests from localhost development servers (React and Vite).

### Middleware Configuration

```csharp
// Configure the HTTP request pipeline
if (app.Environment.IsDevelopment())
{
    // Development-specific middleware
    app.UseDeveloperExceptionPage();
}

// Ensure that HTTPS protocol is used
app.UseHttpsRedirection();

// Use CORS middleware
app.UseCors("AllowLocalhost");

// Add authentication middleware
app.UseAuthentication();
app.UseAuthorization();
```

This configures the HTTP request pipeline with:
- Developer exception page in development
- HTTPS redirection
- CORS middleware
- Authentication and authorization middleware

### Endpoint Mapping

```csharp
// Map API endpoints
app.MapAuthEndpoints();
app.MapRoleEndpoints();
app.MapProductEndpoints();
app.MapOrderEndpoints();

// Add a simple health check endpoint
app.MapGet("/", () => "TinyTreats API is running!");
```

This maps all our API endpoints using the extension methods we defined in our endpoint classes, and adds a simple health check endpoint.

## Creating Database Migrations

Now that we have our models and database context defined, we need to create migrations to set up the database schema. Migrations are a way to incrementally update the database schema to match the data model while preserving existing data.

### Installing the Entity Framework Core Tools

First, make sure you have the Entity Framework Core tools installed:

```bash
dotnet tool install --global dotnet-ef
```

If you already have the tools installed, you can update them to the latest version:

```bash
dotnet tool update --global dotnet-ef
```

### Creating the Initial Migration

To create the initial migration, run the following command in the project directory:

```bash
dotnet ef migrations add InitialCreate
```

This will create a new `Migrations` folder in your project with the initial migration files. These files contain the code to create the database schema based on your models.

If you are on a newer Mac with Apple Silicon, you may need to run this instead:

```bash
arch -arm64 dotnet ef migrations add InitialCreate
```

### Applying the Migration

To apply the migration and create the database, run the following command:

```bash
dotnet ef database update
```

This will create the database and all the tables based on your models.

## Verifying the Database

Verify that the database was created correctly by creating a new connection in your **SQLTools** extension panel that connects to the **TinyTreats** database with the **postgres** user.

Once connected, you should see the following tables:
- `AspNetUsers` (Identity users)
- `AspNetRoles` (Identity roles)
- `AspNetUserRoles` (User-role assignments)
- `AspNetUserClaims` (User claims)
- `AspNetUserLogins` (User logins)
- `AspNetUserTokens` (User tokens)
- `AspNetRoleClaims` (Role claims)
- `UserProfiles` (User profiles)
- `Products` (Products)
- `Orders` (Orders)
- `OrderItems` (Order items)

## Running the Application

Start your API is debug mode.

> 🧨 Make sure you read the output as the port that your API starts on may differ from the one below.

This will start the web server and make your API available at `https://localhost:7086` _(or a similar URL, depending on your configuration)_.

You can test the API using a tool like Yaak, as described in the previous chapters.

## Summary

In this chapter, we've implemented the `Program.cs` file to tie together all the components of our TinyTreats application:

- Database configuration with Entity Framework Core
- Identity configuration for authentication and authorization
- Cookie-based authentication
- Authorization policies
- CORS configuration
- Middleware configuration
- Endpoint mapping

## Next Steps

[Next: Authentication endpoints](./tinytreats-auth-endpoints.md)