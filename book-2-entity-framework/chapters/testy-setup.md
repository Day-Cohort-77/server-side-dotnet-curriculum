# Setting Up the Test Project

## Learning Objectives
- Create a new xUnit test project
- Add essential NuGet packages for integration testing
- Configure the test project to work with our API
- Understand the basic test setup pattern

## Creating the Test Project

Let's create a simple solution with an API project and a test project:

```bash
# Create a new directory for our project
mkdir StudentExamTracker
cd StudentExamTracker

# Create a solution file
dotnet new sln -n StudentExamTracker

# Create the API project
dotnet new webapi -n StudentExamTracker.API

# Create the test project
dotnet new xunit -n StudentExamTracker.Tests

# Add projects to the solution
dotnet sln add StudentExamTracker.API/StudentExamTracker.API.csproj
dotnet sln add StudentExamTracker.Tests/StudentExamTracker.Tests.csproj

# Add a reference from the test project to the API project
dotnet add StudentExamTracker.Tests/StudentExamTracker.Tests.csproj reference StudentExamTracker.API/StudentExamTracker.API.csproj
```

## Adding Essential NuGet Packages

Let's add only the essential packages to our test project:

```bash
cd StudentExamTracker.Tests

# Add Microsoft.AspNetCore.Mvc.Testing for integration testing
dotnet add package Microsoft.AspNetCore.Mvc.Testing

# Add Entity Framework Core in-memory database for testing
dotnet add package Microsoft.EntityFrameworkCore.InMemory

# Add FluentAssertions for readable assertions
dotnet add package FluentAssertions
```

## Creating a Simple Test Factory

Let's create a simplified version of the WebApplicationFactory that configures our application for testing:

```csharp
// StudentExamTracker.Tests/Helpers/TestWebApplicationFactory.cs
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Mvc.Testing;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.DependencyInjection;
using StudentExamTracker.API.Data;
using System.Linq;

namespace StudentExamTracker.Tests.Helpers
{
    public class TestWebApplicationFactory<TProgram>
        : WebApplicationFactory<TProgram> where TProgram : class
    {
        protected override void ConfigureWebHost(IWebHostBuilder builder)
        {
            builder.ConfigureServices(services =>
            {
                // Find and remove the real DbContext registration
                var descriptor = services.SingleOrDefault(
                    d => d.ServiceType == typeof(DbContextOptions<ApplicationDbContext>));

                if (descriptor != null)
                {
                    services.Remove(descriptor);
                }

                // Add in-memory database for testing
                services.AddDbContext<ApplicationDbContext>(options =>
                {
                    options.UseInMemoryDatabase("TestingDb");
                });

                // Create and seed the database
                var sp = services.BuildServiceProvider();
                using (var scope = sp.CreateScope())
                {
                    var db = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
                    db.Database.EnsureCreated();
                }
            });
        }
    }
}
```

## Creating a Simple Base Test Class

Let's create a streamlined base test class with only the essential helper methods:

```csharp
// StudentExamTracker.Tests/Helpers/IntegrationTestBase.cs
using Microsoft.AspNetCore.Mvc.Testing;
using System.Net.Http;
using System.Net.Http.Json;
using System.Threading.Tasks;
using Xunit;

namespace StudentExamTracker.Tests.Helpers
{
    public class IntegrationTestBase : IClassFixture<TestWebApplicationFactory<Program>>
    {
        protected readonly HttpClient _client;

        public IntegrationTestBase(TestWebApplicationFactory<Program> factory)
        {
            _client = factory.CreateClient();
        }

        // Essential helper methods
        protected async Task<T> GetAsync<T>(string url)
        {
            var response = await _client.GetAsync(url);
            response.EnsureSuccessStatusCode();
            return await response.Content.ReadFromJsonAsync<T>();
        }

        protected async Task<HttpResponseMessage> PostAsync<T>(string url, T data)
        {
            return await _client.PostAsJsonAsync(url, data);
        }
    }
}
```

## Creating a Basic Test

Let's create a simple test to verify our setup works:

```csharp
// StudentExamTracker.Tests/BasicTests.cs
using FluentAssertions;
using StudentExamTracker.Tests.Helpers;
using System.Net;
using System.Threading.Tasks;
using Xunit;

namespace StudentExamTracker.Tests
{
    public class BasicTests : IntegrationTestBase
    {
        public BasicTests(TestWebApplicationFactory<Program> factory) : base(factory)
        {
        }

        [Fact]
        public async Task Get_WeatherForecast_ReturnsSuccessStatusCode()
        {
            // Act
            var response = await _client.GetAsync("/weatherforecast");

            // Assert
            response.StatusCode.Should().Be(HttpStatusCode.OK);
        }
    }
}
```

## Understanding the Test Setup

Here's what we've accomplished:

1. Created a solution with an API project and a test project
2. Added essential NuGet packages for integration testing
3. Created a simplified test factory that:
   - Uses an in-memory database instead of a real database
   - Ensures the database is created for testing
4. Created a streamlined base test class with:
   - An HTTP client for making requests to our API
   - Helper methods for the most common operations (GET and POST)
5. Created a basic test to verify our setup works

This minimal setup allows us to:
- Test our API endpoints with an in-memory database
- Make HTTP requests to our API
- Verify that responses meet our expectations

## Next Steps

In the next chapter, we'll create the essential models for our Student Exam Tracker API. We'll focus on the core entities and their relationships.

After that, we'll implement the most critical API endpoints and write focused tests for them.

[Next: Creating the API Models](./testy-models.md)