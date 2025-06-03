# Implementing Equipment Tests

## Learning Objectives
- Understand how to implement integration tests for the Equipment API
- Learn how to test HTTP operations to your API
- Implement the `GetEquipmentById_ReturnsCorrectEquipment` test method
- Understand how to make assertions about API responses

## Introduction

In this chapter, we'll focus on implementing integration tests for the Equipment API. The TestTube project already has a test module for equipment tests (`EquipmentTests.cs`), but one of the test methods is incomplete: `GetEquipmentById_ReturnsCorrectEquipment`.

Let's start by examining the existing tests to understand the patterns, then implement the missing test method.

## Examining Existing Equipment Tests

Open the `EquipmentTests.cs` file in the TestTubes.Tests project. You'll see several test methods that test different aspects of the Equipment API:

1. `GetAllEquipment_ReturnsAllEquipment`: Tests that the GET `/equipment` endpoint returns all equipment items
2. `GetEquipmentById_ReturnsCorrectEquipment`: This is the method we need to implement

Let's look at the `GetAllEquipment_ReturnsAllEquipment` method to understand the pattern:

```csharp
[Fact]
public async Task GetAllEquipment_ReturnsAllEquipment()
{
    // Arrange
    // Nothing to arrange for this test

    // Act
    var equipment = await TestHelper.GetAllEquipmentAsync(_client);

   // Assert
   Assert.NotNull(equipment);
   Assert.Contains(equipment, e => e.Name == "Microscope");
   Assert.Contains(equipment, e => e.Name == "Centrifuge");
   Assert.Contains(equipment, e => e.Name == "Spectrometer");
   Assert.Contains(equipment, e => e.Name == "PCR Machine");
}
```

This test:
1. Gets the HTTP client from the TestHelper
2. Makes a GET request to the `/equipment` endpoint
3. Deserializes the response to a list of Equipment objects
4. Verifies that the response has a 200 OK status code
5. Verifies that the list contains 4 equipment items (which matches the seed data)

## Implementing the Missing Test Method

Your job is to implement the `GetEquipmentById_ReturnsCorrectEquipment()` method. When you can run `dotnet test` and have that test pass, move to the next section to see how to test error cases.

## Testing Error Cases

It's also important to test error cases, such as what happens when you request an equipment item with an invalid ID. Let's add a test for that:

```csharp
[Fact]
public async Task GetEquipmentById_WithInvalidId_ReturnsNotFound()
{
    // Arrange
    var client = _testHelper.Client;
    var invalidId = 999; // An ID that doesn't exist

    // Act
    var response = await client.GetAsync($"/equipment/{invalidId}");

    // Assert
    Assert.Equal(HttpStatusCode.NotFound, response.StatusCode);
}
```

This test:
1. Defines an invalid ID (one that doesn't exist in the seed data)
2. Makes a GET request to the `/equipment/999` endpoint
3. Verifies that the response has a 404 Not Found status code

## Running the Tests

Now that we've implemented the missing test method, let's run the tests to make sure they pass:

```bash
cd TestTubes.Tests
dotnet test
```

If everything is implemented correctly, both of the equipment tests should be passing.

## Next Steps

Now that we've implemented the equipment tests, let's move on to implementing the scientist tests.

[Implementing Scientist Tests](./testtube-scientist-tests.md)