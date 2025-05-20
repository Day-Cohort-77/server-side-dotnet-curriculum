# Tiny Treats Bakery Shoppe

## Introduction to Authentication and Authorization

In this project, we'll introduce the concepts of authentication and authorization in ASP.NET Core applications. We'll also discuss how we'll organize our authentication endpoints using a clean, maintainable approach.

## Understanding Authentication and Authorization

Before we dive into the implementation, let's clarify the difference between authentication and authorization:

- **Authentication** is the process of verifying who a user is. It answers the question, "Who are you?"
- **Authorization** is the process of verifying what a user is allowed to do. It answers the question, "What can you do?"

In a typical web application, authentication happens first (the user logs in), and then authorization happens for each request (the application checks if the user has permission to access a resource).

## ASP.NET Core Identity

ASP.NET Core Identity is a membership system that adds login functionality to ASP.NET Core apps. It provides:

- User authentication
- Password hashing and validation
- User data storage
- Role-based authorization
- External login providers (like Google, Facebook, etc.)
- Two-factor authentication

We'll use ASP.NET Core Identity to implement authentication and authorization in our application.

## Project Structure

In our approach, we'll organize our authentication endpoints, models, and DTOs in separate files, following the pattern of separating endpoints by functionality. This approach offers several benefits:

1. **Better organization** - Authentication endpoints are grouped together, making them easier to find and modify.
2. **Improved maintainability** - The `Program.cs` file remains clean and focused on configuration.
3. **Scalability** - As your application grows, you can add new endpoint classes without cluttering `Program.cs`.
4. **Testability** - Endpoint classes can be tested independently.

Here's how we'll structure our project:

```
TinyTreats/
├── Endpoints/
│   ├── AuthEndpoints.cs
│   ├── RoleEndpoints.cs
│   └── OrderEndpoints.cs
├── Models/
│   ├── UserProfile.cs
│   └── Order.cs
│   └── OrderItem.cs
│   └── Product.cs
├── Data/
│   └── TinyTreatsDbContext.cs
|── DTO
│   ├── AuthDTOs.cs
│   └── OrderDTOs.cs
│   └── ProductDTOs.cs
│   └── RoleDTOs.cs
├── Program.cs
└── ...
```

This approach keeps our code clean, maintainable, and scalable.

## What We'll Build

In the following chapters, we'll build a simple bakery management system called "TinyTreats" with the following features:

1. **User Registration and Login** - Users can register and log in to the application.
2. **Role-Based Authorization** - Different users have different roles (Admin, Baker, Customer) with different permissions.
3. **Order Management** - Users can create orders, and staff can manage them.

We'll implement these features using ASP.NET Core Identity and Minimal API, with a focus on clean, maintainable code organization.

## Project Setup

### Get the client

Visit the [Tiny Treats client repo](https://github.com/nashville-software-school/tinytreats-client) and click the **Use this template** button to get a copy of the project for your Github account.

### Start the API

Let's create a new project called "TinyTreats" - a simple bakery management system where we'll implement authentication.

1. Create a new ASP.NET Core Minimal API project in the directory of your choice.
   ```bash
   dotnet new web -n TinyTreats
   cd TinyTreats
   ```

2. Inside the `TinyTreats` directory, run:
   ```bash
   dotnet new gitignore
   ```

3. Add the required NuGet packages:
   ```bash
   dotnet add package Microsoft.EntityFrameworkCore.Design --version 8.0.0
   dotnet add package Microsoft.EntityFrameworkCore.Tools --version 8.0.0
   dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL --version 8.0.0
   dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore --version 8.0.0
   ```

4. Set up user secrets for database connection:
   ```bash
   dotnet user-secrets init
   dotnet user-secrets set "TinyTreatsDbConnectionString" "Host=localhost;Port=5432;Username=postgres;Password=<your-password>;Database=TinyTreats"
   ```

5. Create the necessary folders for our organized project structure:
   ```bash
   mkdir Models Data Endpoints DTO
   ```

6. Refer to the [debugging chapter](../../book-1-csharp-sql/chapters/debugging-csharp.md) to create your `.vscode/launch.json` and `.vscode/tasks.json`. Replace all instances of **HarborMaster** with **TinyTreats**.

## Next Steps

In the next chapter, we'll set up our project and implement user registration and login functionality.

[Next: Setting up the models](./tinytreats-models-dtos.md)