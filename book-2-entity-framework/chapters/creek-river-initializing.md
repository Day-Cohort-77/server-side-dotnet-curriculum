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
   dotnet user-secrets set 'ConnectionStrings:CreekRiverDbConnectionString' 'Host=localhost;Port=5432;Username=postgres;Password=<your_postgresql_password>;Database=CreekRiver'
   ```

6. Refer to the [debugging chapter](../../book-1-csharp-sql/chapters/debugging-csharp.md) to create your `.vscode/launch.json` and `.vscode/tasks.json`. Replace all instances of **HarborMaster** with **CreekRiver**.

7. Create a `Models` directory in your project root to organize your data models:
   ```bash
   mkdir Models Services Endpoints
   ```

## Project Structure

Let's take a moment to understand what we've created:

1. **Web API Project**: We've created a minimal API project, which is a lightweight approach to building APIs with ASP.NET Core.

2. **Entity Framework Core Packages**:
   - `Npgsql.EntityFrameworkCore.PostgreSQL`: This package allows EF Core to work with PostgreSQL databases.
   - `Microsoft.EntityFrameworkCore.Design`: This package provides design-time support for EF Core migrations.

## User Secrets


User Secrets is a feature in ASP.NET Core that allows developers to store sensitive information, such as database connection strings, API keys, and other configuration data, outside of the source code. This is particularly useful during development to avoid hardcoding sensitive information in the codebase, which could be accidentally committed to version control systems.

**Purpose**:
- **Security**: By storing secrets separately, you reduce the risk of exposing sensitive information in your source code.
- **Environment-Specific Configuration**: User Secrets allow you to manage different configurations for different environments (development, staging, production) without changing the code.
- **Ease of Use**: It provides a simple way to manage secrets during development without requiring complex configuration management tools.

**Usage**:
1. **Initialization**: You initialize User Secrets in your project using the command `dotnet user-secrets init`. This creates a unique identifier for your project in a `UserSecretsId` property in your `.csproj` file.
2. **Setting Secrets**: You can set secrets using the command `dotnet user-secrets set <key> <value>`. For example, setting a connection string as shown in the previous steps.
3. **Accessing Secrets**: In your application, you can access these secrets through the configuration system. For example, using `Configuration.GetConnectionString("CreekRiverDbConnectionString")` to retrieve the connection string.

**Storage**:
- User Secrets are stored in a JSON file located in a directory specific to your user profile on your machine. The path typically looks like this:
  - Windows: `%APPDATA%\Microsoft\UserSecrets\<user_secrets_id>\secrets.json`
  - macOS/Linux: `~/.microsoft/usersecrets/<user_secrets_id>/secrets.json`
- This file is not checked into source control, ensuring that sensitive information remains private.

By using User Secrets, developers can maintain a secure and flexible configuration management strategy during the development phase of their applications.

## What's Next?

In the next chapter, we'll create our data models and set up the database structure. These models will serve two purposes:
1. Define the types we'll use in our .NET application
2. Provide EF Core with the information it needs to create the corresponding database tables

Up Next: [Setting up the database](./creek-river-models.md)

## 🔍 Additional Materials

- [Entity Framework Core Documentation](https://docs.microsoft.com/en-us/ef/core/)
- [ASP.NET Core Minimal APIs](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis)
- [Safe storage of app secrets in development in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/security/app-secrets)
