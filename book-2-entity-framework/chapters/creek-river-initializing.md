# Initializing a new EF API project

In this chapter, we'll set up a new API project that uses Entity Framework Core to interact with a PostgreSQL database. Entity Framework Core (EF Core) is an object-relational mapper (ORM) that allows .NET developers to work with a database using .NET objects, eliminating the need to write most of the data-access code.

## Creating the Project

1. In the csharp directory of your workspace create the web api project like this:
   ```bash
   dotnet new web -n CreekRiver
   cd CreekRiver
   ```

2. Inside the `CreekRiver` directory, run:
   ```bash
   dotnet new gitignore
   ```

3. Install these required dependencies:
   ```bash
   dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL --version 8.0
   ```
   and:
   ```bash
   dotnet add package Microsoft.EntityFrameworkCore.Design --version 8.0
   ```

4. Run this to be able to store secrets for this app:
   ```bash
   dotnet user-secrets init
   ```

5. Add the connection string to the secrets for this app (make sure you change `<your_postgresql_password>` to your db password!):
   ```bash
   dotnet user-secrets set 'CreekRiverDbConnectionString' 'Host=localhost;Port=5432;Username=postgres;Password=<your_postgresql_password>;Database=CreekRiver'
   ```

6. Refer to the [debugging chapter](../../book-1-csharp-sql/chapters/debugging-csharp.md) to create your `.vscode/launch.json` and `.vscode/tasks.json`. Replace all instances of **HarborMaster** with **CreekRiver**.

## Disable HTTPS Redirection

By default, a minimal .NET Core API forces all incoming HTTP requests to switch to the HTTPS protocol, which you don't need for these project. It's easy to disable.

1. Open your project and open the `Program.cs` file.
2. Find the `app.UseHttpsRedirection();` line of code.
3. Delete it

Done.

## Project Structure

Let's take a moment to understand what we've created:

1. **Web API Project**: We've created a minimal API project, which is a lightweight approach to building APIs with ASP.NET Core.

2. **Entity Framework Core Packages**:
   - `Npgsql.EntityFrameworkCore.PostgreSQL`: This package allows EF Core to work with PostgreSQL databases.
   - `Microsoft.EntityFrameworkCore.Design`: This package provides design-time support for EF Core migrations.

3. **User Secrets**: We've initialized user secrets for our project and stored our database connection string securely. This is a best practice for managing sensitive information like connection strings.

## What's Next?

In the next chapter, we'll create our data models and set up the database structure. These models will serve two purposes:
1. Define the types we'll use in our .NET application
2. Provide EF Core with the information it needs to create the corresponding database tables

Up Next: [Setting up the database](./creek-river-models.md)

## 🔍 Additional Materials

- [Entity Framework Core Documentation](https://docs.microsoft.com/en-us/ef/core/)
- [ASP.NET Core Minimal APIs](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis)
- [Safe storage of app secrets in development in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/security/app-secrets)

## 🔍 Additional Materials

- [Entity Framework Core Documentation](https://docs.microsoft.com/en-us/ef/core/)
- [ASP.NET Core Minimal APIs](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis)
- [Safe storage of app secrets in development in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/security/app-secrets)