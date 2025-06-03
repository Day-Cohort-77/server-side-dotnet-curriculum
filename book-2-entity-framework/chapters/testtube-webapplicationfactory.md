# Understanding WebApplicationFactory

## Learning Objectives
- Understand what WebApplicationFactory is and why it's important for integration testing
- Learn how WebApplicationFactory configures a test server
- Understand how to customize WebApplicationFactory for testing
- Learn how to use WebApplicationFactory to make HTTP requests to your API

## What is WebApplicationFactory?

WebApplicationFactory is a class provided by the `Microsoft.AspNetCore.Mvc.Testing` package that helps you create a test host for your ASP.NET Core application. This test host allows you to make HTTP requests to your API without having to start a real HTTP server, making your tests faster and more reliable.

The WebApplicationFactory creates an in-memory server that hosts your API, configures services (like database connections), and provides an HTTP client that you can use to make requests to your API.

## Why Use WebApplicationFactory?

There are several benefits to using WebApplicationFactory for integration testing:

1. **Full application startup**: It runs through the entire application startup process, including middleware configuration.
2. **In-memory hosting**: No need to start a real HTTP server, making tests faster.
3. **Dependency injection**: You can override services in the application's service collection.
4. **Isolated testing**: Each test can have its own instance of the application.
5. **HTTP client**: It provides an HTTP client configured to work with your application.

## How WebApplicationFactory Works

When you create a WebApplicationFactory, it:

1. Finds your application's entry point (Program.cs)
2. Creates a test server using your application's configuration
3. Configures services, including replacing real services with test doubles if needed
4. Provides an HTTP client that's configured to work with the test server

Here's a simplified example of how to use WebApplicationFactory:

```csharp
public class MyApiTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly WebApplicationFactory<Program> _factory;

    public MyApiTests(WebApplicationFactory<Program> factory)
    {
        _factory = factory;
    }

    [Fact]
    public async Task GetEndpoint_ReturnsSuccessStatusCode()
    {
        // Arrange
        var client = _factory.CreateClient();

        // Act
        var response = await client.GetAsync("/myendpoint");

        // Assert
        response.EnsureSuccessStatusCode();
    }
}
```

In this example:
- We're using xUnit's `IClassFixture` to create a WebApplicationFactory for our tests
- The factory is injected into our test class constructor
- We create an HTTP client using the factory
- We use the client to make a GET request to our API
- We assert that the response has a success status code

## Customizing WebApplicationFactory

One of the most powerful features of WebApplicationFactory is the ability to customize it for your tests. You can override services, configure the host, and more.

For example, you might want to replace the real database with an in-memory database for testing:

```csharp
public class CustomWebApplicationFactory<TProgram> : WebApplicationFactory<TProgram> where TProgram : class
{
    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.ConfigureServices(services =>
        {
            // Remove the real database context registration
            var descriptor = services.SingleOrDefault(
                d => d.ServiceType == typeof(DbContextOptions<MyDbContext>));

            if (descriptor != null)
            {
                services.Remove(descriptor);
            }

            // Add the in-memory database context
            services.AddDbContext<MyDbContext>(options =>
            {
                options.UseInMemoryDatabase("TestDatabase");
            });

            // Seed the database with test data
            var sp = services.BuildServiceProvider();
            using var scope = sp.CreateScope();
            var scopedServices = scope.ServiceProvider;
            var db = scopedServices.GetRequiredService<MyDbContext>();
            db.Database.EnsureCreated();

            // Add test data here
            db.MyEntities.Add(new MyEntity { Id = 1, Name = "Test Entity" });
            db.SaveChanges();
        });
    }
}
```

In this example:
- We create a custom WebApplicationFactory that inherits from the base WebApplicationFactory
- We override the ConfigureWebHost method to customize the services
- We remove the real database context registration
- We add an in-memory database context
- We seed the database with test data

## WebApplicationFactory in the TestTube Project

In the TestTube project, we'll use a custom WebApplicationFactory to:

1. Configure the application to use the Testing environment
2. Use an in-memory database for testing
3. Seed the database with test data for our tests

Here's the implementation of our `TestWebApplicationFactory` class:

```cs
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Mvc.Testing;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using TestTube.Data;

namespace TestTube.Tests;

public class TestWebApplicationFactory : WebApplicationFactory<Program>
{
    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        // Set the environment to Testing
        builder.UseEnvironment("Testing");

        // The Program.cs will now use the in-memory database for the Testing environment

        builder.ConfigureServices(services =>
        {
            // Build the service provider
            var sp = services.BuildServiceProvider();

            // Create a scope to obtain a reference to the database context
            using var scope = sp.CreateScope();
            var scopedServices = scope.ServiceProvider;
            var db = scopedServices.GetRequiredService<ApplicationDbContext>();
            var logger = scopedServices.GetRequiredService<ILogger<TestWebApplicationFactory>>();

            try
            {
                // Ensure the database is created
                db.Database.EnsureCreated();

                // Seed the database with test data
                TestHelper.SeedDatabase(db);
            }
            catch (Exception ex)
            {
                logger.LogError(ex, "An error occurred seeding the database. Error: {Message}", ex.Message);
            }
        });
    }
}
```

This implementation:

1. Sets the environment to "Testing", which allows our application to use different configuration for tests
2. Uses the service provider to get access to the database context
3. Creates the database and seeds it with test data using the `TestHelper.SeedDatabase` method
4. Includes error handling to log any issues that occur during database setup

This will allow us to write integration tests that verify the behavior of our API endpoints without needing a real database.

In the next chapter, we'll explore the TestHelper class, which uses WebApplicationFactory to set up our test environment.

## Next Steps

Now that we understand WebApplicationFactory, let's look at how it's used in the TestHelper class to set up our test environment.

[The TestHelper Class](./testtube-testhelper.md)