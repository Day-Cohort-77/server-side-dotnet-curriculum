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
3. `CreateEquipment_AddsNewEquipment`: Tests that the POST `/equipment` endpoint adds a new equipment item
4. `UpdateEquipment_ModifiesExistingEquipment`: Tests that the PUT `/equipment/{id}` endpoint updates an existing equipment item
5. `DeleteEquipment_RemovesEquipment`: Tests that the DELETE `/equipment/{id}` endpoint removes an equipment item

Let's look at the `GetAllEquipment_ReturnsAllEquipment` method to understand the pattern:

```csharp
[Fact]
public async Task GetAllEquipment_ReturnsAllEquipment()
{
    // Arrange
    var client = _testHelper.Client;

    // Act
    var response = await client.GetAsync("/equipment");
    var equipment = await response.Content.ReadFromJsonAsync<List<Equipment>>();

    // Assert
    Assert.Equal(HttpStatusCode.OK, response.StatusCode);
    Assert.Equal(4, equipment.Count);
}
```

This test:
1. Gets the HTTP client from the TestHelper
2. Makes a GET request to the `/equipment` endpoint
3. Deserializes the response to a list of Equipment objects
4. Verifies that the response has a 200 OK status code
5. Verifies that the list contains 4 equipment items (which matches the seed data)

Now let's look at how the `CreateEquipment_AddsNewEquipment` method would be implemented when the API supports POST operations to the `/equipment` endpoint _(which it doesn't yet)_.

```csharp
[Fact]
public async Task CreateEquipment_AddsNewEquipment()
{
    // Arrange
    var client = _testHelper.Client;
    var newEquipment = new Equipment
    {
        Name = "Test Equipment",
        Type = "Test Type",
        ScientistId = 1
    };

    // Act
    var response = await client.PostAsJsonAsync("/equipment", newEquipment);
    var createdEquipment = await response.Content.ReadFromJsonAsync<Equipment>();

    // Assert
    Assert.Equal(HttpStatusCode.Created, response.StatusCode);
    Assert.Equal(newEquipment.Name, createdEquipment.Name);
    Assert.Equal(newEquipment.Type, createdEquipment.Type);
    Assert.Equal(newEquipment.ScientistId, createdEquipment.ScientistId);
    Assert.True(createdEquipment.Id > 0);

    // Verify the equipment was actually added
    var getAllResponse = await client.GetAsync("/equipment");
    var allEquipment = await getAllResponse.Content.ReadFromJsonAsync<List<Equipment>>();
    Assert.Equal(5, allEquipment.Count); // 4 seed items + 1 new item
}
```

This test:
1. Creates a new Equipment object
2. Makes a POST request to the `/equipment` endpoint with the new equipment
3. Deserializes the response to an Equipment object
4. Verifies that the response has a 201 Created status code
5. Verifies that the created equipment has the expected properties
6. Makes a GET request to verify that the equipment was actually added to the database

## Implement the POST Method

Open the `Endpoints/EquipmentEndpoints.cs` module in the API project and implement a method that allows a client to perform a POST operation to add a piece of equipment and test it out with Yaak.

Then, run `dotnet test` and see if the test passes.

> 📝 This is called **Test-Driven Development**. When you develop the test first, you complete the algorithmic thinking process and then you know exactly what code to write in the implementation of the feature.

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

If everything is implemented correctly, all tests should pass.

## Next Steps

Now that we've implemented the equipment tests, let's move on to implementing the scientist tests.

[Implementing Scientist Tests](./testtube-scientist-tests.md)