# The TestHelper Class

## Learning Objectives
- Understand the purpose of the TestHelper class in integration testing
- Learn how TestHelper seeds the database with test data
- Understand how TestHelper provides methods for HTTP client operations
- Learn how to use TestHelper in your tests

## What is the TestHelper Class?

The TestHelper class is a static utility class that simplifies integration testing by providing methods to seed the database with test data and perform common HTTP client operations. It works with the in-memory database configuration that's automatically set up when the application runs in the "Testing" environment.

In the TestTube project, the TestHelper class is already implemented for you. Let's examine it to understand how it works.

## Examining the TestHelper Class

Open the `TestHelper.cs` file in the TestTube.Tests project. You'll see something like this:

```csharp
using System.Text.Json;
using TestTube.Data;
using TestTube.DTOs;
using TestTube.Models;

namespace TestTube.Tests;

public static class TestHelper
{
    public static void SeedDatabase(ApplicationDbContext dbContext)
    {
        // Add scientists
        var scientist1 = new Scientist
        {
            Id = 1,
            Name = "Marie Curie",
            Department = "Physics",
            Email = "marie.curie@testtube.com",
            HireDate = new DateTime(2020, 1, 15)
        };

        var scientist2 = new Scientist
        {
            Id = 2,
            Name = "Albert Einstein",
            Department = "Physics",
            Email = "albert.einstein@testtube.com",
            HireDate = new DateTime(2021, 3, 10)
        };

        var scientist3 = new Scientist
        {
            Id = 3,
            Name = "Rosalind Franklin",
            Department = "Chemistry",
            Email = "rosalind.franklin@testtube.com",
            HireDate = new DateTime(2022, 5, 20)
        };

        dbContext.Scientists.AddRange(scientist1, scientist2, scientist3);
        dbContext.SaveChanges();

        // Add equipment
        var equipment1 = new Equipment
        {
            Id = 1,
            Name = "Microscope",
            SerialNumber = "MS-12345",
            Manufacturer = "Zeiss",
            PurchaseDate = new DateTime(2021, 2, 10),
            Price = 15000.00m,
            ScientistId = 1
        };

        var equipment2 = new Equipment
        {
            Id = 2,
            Name = "Centrifuge",
            SerialNumber = "CF-67890",
            Manufacturer = "Thermo Fisher",
            PurchaseDate = new DateTime(2022, 4, 15),
            Price = 8500.00m,
            ScientistId = 1
        };

        var equipment3 = new Equipment
        {
            Id = 3,
            Name = "Spectrometer",
            SerialNumber = "SP-24680",
            Manufacturer = "Agilent",
            PurchaseDate = new DateTime(2023, 1, 5),
            Price = 22000.00m,
            ScientistId = 2
        };

        var equipment4 = new Equipment
        {
            Id = 4,
            Name = "PCR Machine",
            SerialNumber = "PCR-13579",
            Manufacturer = "Bio-Rad",
            PurchaseDate = new DateTime(2023, 6, 20),
            Price = 12000.00m,
            ScientistId = 3
        };

        dbContext.Equipment.AddRange(equipment1, equipment2, equipment3, equipment4);
        dbContext.SaveChanges();
    }

    // HTTP client methods for integration testing
    public static async Task<List<ScientistDto>> GetAllScientistsAsync(HttpClient client)
    {
        var response = await client.GetAsync("/scientists");
        response.EnsureSuccessStatusCode();
        var content = await response.Content.ReadAsStringAsync();
        return JsonSerializer.Deserialize<List<ScientistDto>>(content, new JsonSerializerOptions
        {
            PropertyNameCaseInsensitive = true
        });
    }

    public static async Task<ScientistDetailDto> GetScientistByIdAsync(HttpClient client, int id)
    {
        var response = await client.GetAsync($"/scientists/{id}");
        response.EnsureSuccessStatusCode();
        var content = await response.Content.ReadAsStringAsync();
        return JsonSerializer.Deserialize<ScientistDetailDto>(content, new JsonSerializerOptions
        {
            PropertyNameCaseInsensitive = true
        });
    }

    public static async Task<List<EquipmentDto>> GetAllEquipmentAsync(HttpClient client)
    {
        var response = await client.GetAsync("/equipment");
        response.EnsureSuccessStatusCode();
        var content = await response.Content.ReadAsStringAsync();
        return JsonSerializer.Deserialize<List<EquipmentDto>>(content, new JsonSerializerOptions
        {
            PropertyNameCaseInsensitive = true
        });
    }

    public static async Task<EquipmentDetailDto> GetEquipmentByIdAsync(HttpClient client, int id)
    {
        var response = await client.GetAsync($"/equipment/{id}");
        response.EnsureSuccessStatusCode();
        var content = await response.Content.ReadAsStringAsync();
        return JsonSerializer.Deserialize<EquipmentDetailDto>(content, new JsonSerializerOptions
        {
            PropertyNameCaseInsensitive = true
        });
    }
}
```

Let's break down the key components of this class:

### Static Class Design

The `TestHelper` is now a static class, which means:

1. It doesn't need to be instantiated - you can call its methods directly
2. It doesn't maintain any instance state between calls
3. It's designed to work with the in-memory database that's configured in the `Program.cs` file

### SeedDatabase Method

The `SeedDatabase` method populates the in-memory database with test data:

1. It adds three scientists with detailed information (name, department, email, hire date)
2. It adds four pieces of equipment with detailed information (name, serial number, manufacturer, purchase date, price)
3. It assigns each piece of equipment to a scientist
4. It saves the changes to the database

This ensures that our tests start with a known state, making them more reliable and predictable.

### HTTP Client Helper Methods

The class provides several helper methods for making HTTP requests to the API:

1. `GetAllScientistsAsync` - Gets a list of all scientists
2. `GetScientistByIdAsync` - Gets a specific scientist by ID
3. `GetAllEquipmentAsync` - Gets a list of all equipment
4. `GetEquipmentByIdAsync` - Gets a specific piece of equipment by ID

These methods handle the HTTP request, ensure a successful response, and deserialize the JSON response into the appropriate DTO objects.

## Why Use a TestHelper Class?

There are several benefits to using a TestHelper class:

1. **Encapsulation**: It encapsulates the complexity of setting up the test environment
2. **Reusability**: It can be reused across multiple test classes
3. **Consistency**: It ensures that all tests start with the same database state
4. **Simplicity**: It makes your test classes cleaner and more focused on testing behavior

## Customizing TestHelper for Your Tests

While the TestHelper class provided in the TestTube project is sufficient for most tests, you might want to customize it for specific test scenarios. For example, you might want to:

1. Add different test data to the `SeedDatabase` method
2. Add new helper methods for additional API endpoints
3. Add methods for handling authentication tokens
4. Create specialized DTO classes for specific test scenarios

## Next Steps

Now that we understand the TestHelper class, let's look at how test modules are structured in the TestTube project.

[Anatomy of a Test Module](./testtube-test-module.md)