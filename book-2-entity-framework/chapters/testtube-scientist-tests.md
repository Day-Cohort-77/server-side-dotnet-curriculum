# Implementing Scientist Tests

## Learning Objectives
- Understand how to implement integration tests for the Scientist API
- Learn how to test relationships between entities
- Implement the `GetAllScientists_ReturnsAllScientists` test method
- Understand how to test filtering and querying in API endpoints

## Introduction

In this chapter, we'll focus on implementing integration tests for the Scientist API. The TestTube project already has a test module for scientist tests (`ScientistTests.cs`), but one of the test methods is incomplete: `GetAllScientists_ReturnsAllScientists`.

Let's start by examining the existing tests to understand the patterns, then implement the missing test method.

## Examining Existing Scientist Tests

Open the `ScientistTests.cs` file in the TestTubes.Tests project. You'll see several test methods that test different aspects of the Scientist API:

1. `GetAllScientists_ReturnsAllScientists`: This is the method we need to implement
2. `GetScientistById_ReturnsCorrectScientist`: Tests that the GET `/scientist/{id}` endpoint returns the correct scientist
3. `GetScientistWithEquipment_IncludesEquipment`: Tests that the GET `/scientist/{id}` endpoint includes the scientist's equipment
4. `CreateScientist_AddsNewScientist`: Tests that the POST `/scientist` endpoint adds a new scientist

Let's look at the `GetScientistById_ReturnsCorrectScientist` method to understand the pattern:

```csharp
[Fact]
public async Task GetScientistById_ReturnsCorrectScientist()
{
    // Arrange
    var client = _testHelper.Client;
    var expectedId = 1;
    var expectedName = "Marie Curie";
    var expectedSpecialty = "Radioactivity";

    // Act
    var response = await client.GetAsync($"/scientists/{expectedId}");
    var scientist = await response.Content.ReadFromJsonAsync<Scientist>();

    // Assert
    Assert.Equal(HttpStatusCode.OK, response.StatusCode);
    Assert.Equal(expectedId, scientist.Id);
    Assert.Equal(expectedName, scientist.Name);
    Assert.Equal(expectedSpecialty, scientist.Specialty);
}
```

This test:
1. Gets the HTTP client from the TestHelper
2. Defines the expected values for the scientist with ID 1 (which is part of the seed data)
3. Makes a GET request to the `/scientists/1` endpoint
4. Deserializes the response to a Scientist object
5. Verifies that the response has a 200 OK status code
6. Verifies that the returned scientist has the expected properties

## Implementing the Missing Test Method

Your job is to implement the `GetAllScientists_ReturnsAllScientists()` method.

## Running the Tests

Once you have implemented the missing test method, let's run the tests to make sure they pass:

```bash
cd TestTubes.Tests
dotnet test
```

If everything is implemented correctly, all tests should pass.

## Optional Learning

If you want to build your competency with implementing API endpoints, and their corresponding tests, you can work on the following tasks.

1. Implement a `MapPut()` method in your scientist endpoint in your API project that will support PUT operations to `/scientists`.
2. Then implement a test method named `UpdateScientist_ModifiesExistingScientist` which tests that the PUT `/scientist/{id}` endpoint updates an existing scientist
3. Implement a `MapDelete()` method in your scientist endpoint in your API project that will support DELETE operations to `/scientists/{id}`.
4. Implement a test method named `DeleteScientist_RemovesScientist` which tests that the DELETE `/scientist/{id}` endpoint removes a scientist


## Next Steps

Now that we've implemented both the equipment and scientist tests, let's look at how to measure and improve test coverage.

[Achieving Test Coverage](./testtube-test-coverage.md)