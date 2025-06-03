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

Let's look at the `GetScientistById_ReturnsCorrectScientist` method to understand the pattern:

```csharp
 [Fact]
 public async Task GetScientistById_ReturnsCorrectScientist()
 {
   // Arrange
   int scientistId = 1;

   // Act
   var scientist = await TestHelper.GetScientistByIdAsync(_client, scientistId);

   // Assert
   Assert.NotNull(scientist);
   Assert.Equal(scientistId, scientist.Id);
   Assert.Equal("Marie Curie", scientist.Name);
   Assert.Equal("Physics", scientist.Department);
   Assert.NotNull(scientist.Equipment);
   // We don't check for a specific count as the number of equipment items can vary

   // Check equipment details
   var equipment = scientist.Equipment.ToList();
   Assert.Contains(equipment, e => e.Name == "Microscope");
   Assert.Contains(equipment, e => e.Name == "Centrifuge");
}

```

This test:

1. Defines the expected values for the scientist with ID 1 (which is part of the seed data)
2. Makes a GET request to the `/scientists/1` endpoint by using the method from the `TestHelper`.
3. Deserializes the response to a Scientist object
5. Verifies that the returned scientist has the expected property values

## Implementing the Missing Test Method

Your job is to implement the `GetAllScientists_ReturnsAllScientists()` method. Refer to the equipment tests module for help you out.

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