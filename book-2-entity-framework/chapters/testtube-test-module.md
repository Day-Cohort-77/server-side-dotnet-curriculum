# Anatomy of a Test Module

## Learning Objectives
- Understand the structure of a test module in xUnit
- Learn how to organize tests within a module
- Understand the Arrange-Act-Assert pattern
- Learn how to write effective test methods

## What is a Test Module?

In the context of our TestTube project, a test module is a class that contains related test methods. Each test module typically focuses on testing a specific component or feature of the application. In our case, we have two test modules:

1. `EquipmentTests.cs`: Contains tests for the equipment endpoints
2. `ScientistTests.cs`: Contains tests for the scientist endpoints

Let's examine the structure of a test module to understand how it's organized.

## Test Module Structure

Open the `EquipmentTests.cs` file in the TestTubes.Tests project. In this module, there is one implemented test, and another one that you will implement later.

```cs
namespace TestTube.Tests;

public class EquipmentTests : IClassFixture<TestWebApplicationFactory>, IDisposable
{
    private readonly TestWebApplicationFactory _factory;
    private readonly HttpClient _client;

    public EquipmentTests(TestWebApplicationFactory factory)
    {
        _factory = factory;
        _client = _factory.CreateClient();
    }

    [Fact]
    public async Task GetAllEquipment_ReturnsAllEquipment()
    {
        // Act
        var equipment = await TestHelper.GetAllEquipmentAsync(_client);

        // Assert
        Assert.NotNull(equipment);
        // We don't check for a specific count as the number of items can vary from 0 to n
        Assert.Contains(equipment, e => e.Name == "Microscope");
        Assert.Contains(equipment, e => e.Name == "Centrifuge");
        Assert.Contains(equipment, e => e.Name == "Spectrometer");
        Assert.Contains(equipment, e => e.Name == "PCR Machine");
    }

    [Fact]
    public async Task GetEquipmentById_ReturnsCorrectEquipment()
    {
        // Arrange
        int equipmentId = 1;

        // Act

        // Assert
    }

    public void Dispose()
    {
        _client.Dispose();
    }
}
```

Let's break down the key components of this test module:

### Namespace and Class Declaration

The test module is defined as a class within the `TestTube.Tests` namespace. The class name typically reflects what's being tested, in this case, `EquipmentTests`.

### Interface Implementation and Constructor

The test module implements two interfaces and has a constructor that initializes the necessary components:

```cs
public class EquipmentTests : IClassFixture<TestWebApplicationFactory>, IDisposable
{
    private readonly TestWebApplicationFactory _factory;
    private readonly HttpClient _client;

    public EquipmentTests(TestWebApplicationFactory factory)
    {
        _factory = factory;
        _client = _factory.CreateClient();
    }
```

Let's break down what's happening here:

1. `IClassFixture<TestWebApplicationFactory>`: This is an xUnit interface that tells the test runner to create a single instance of `TestWebApplicationFactory` and provide it to all test methods in this class. This ensures that all tests share the same test server and database.

2. `IDisposable`: This interface allows the test class to clean up resources when it's done. In this case, we need to dispose of the HTTP client.

3. The constructor takes a `TestWebApplicationFactory` parameter, which is injected by xUnit. It then:
   - Stores the factory for later use
   - Creates an HTTP client that's configured to work with the test server

### Test Methods

The test module contains one or more test methods, each annotated with the `[Fact]` attribute:

```cs
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
}
```

Each test method tests a specific aspect of the application's behavior. The method name typically follows the pattern `MethodName_ExpectedBehavior`, which makes it clear what's being tested and what the expected outcome is.

### Resource Cleanup

Since the test class implements `IDisposable`, it includes a `Dispose` method to clean up resources:

```cs
public void Dispose()
{
    _client.Dispose();
}
```

This ensures that the HTTP client is properly disposed of after the tests are complete, preventing resource leaks.

## The Arrange-Act-Assert Pattern

Most test methods follow the Arrange-Act-Assert (AAA) pattern, which divides the test into three distinct phases:

1. **Arrange**: Set up the test environment and prepare the inputs
2. **Act**: Perform the action being tested
3. **Assert**: Verify that the action produced the expected result

Let's look at that example again:

```cs
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
}
```

In this example:
- **Arrange**: Sets up the data needed for the test by specifiying `1` as the ID you will be retrieving
- **Act**: We call the `TestHelper.GetScientistByIdAsync` method, which makes a GET request to the `/scientists` endpoint and deserializes the response
- **Assert**: We verify that the response is not null and that the serialized object has all of the correct property values

The AAA pattern makes tests easier to read and understand, as it clearly separates the setup, the action, and the verification.

## Test Method Naming

Good test method names are crucial for understanding what's being tested. A common naming convention is:

```
MethodName_Scenario_ExpectedBehavior
```

For example:
- `GetAllEquipment_ReturnsAllEquipment`: Tests that the GetAllEquipment method returns all equipment
- `GetEquipmentById_WithValidId_ReturnsCorrectEquipment`: Tests that the GetEquipmentById method returns the correct equipment when given a valid ID
- `GetEquipmentById_WithInvalidId_ReturnsNotFound`: Tests that the GetEquipmentById method returns a 404 Not Found when given an invalid ID

Clear and descriptive test method names serve as documentation for your code and make it easier to understand what's being tested.

## Running the Tests

Time for you to run your tests for the first time.

```bash
cd TestTubes.Tests
dotnet test
```

You will notice that some passed, and some failed, because right now, not all tests have been implemented.

## Next Steps

Now that we understand the structure of a test module, let's implement the equipment tests.

[Implementing Equipment Tests](./testtube-equipment-tests.md)