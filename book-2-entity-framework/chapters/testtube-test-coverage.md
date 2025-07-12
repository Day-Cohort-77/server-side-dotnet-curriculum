# Achieving Test Coverage

## Learning Objectives
- Understand what test coverage is and why it's important
- Learn how to measure test coverage in a .NET project
- Identify areas of code that need more testing
- Implement strategies to improve test coverage

## What is Test Coverage?

Test coverage is a measure of how much of your code is executed when your tests run. It's typically expressed as a percentage of lines, branches, or methods that are executed during testing.

High test coverage doesn't guarantee that your code is bug-free, but it does indicate that most of your code has been exercised by your tests. Low test coverage suggests that there are parts of your code that haven't been tested, which increases the risk of undiscovered bugs.

## Why Test Coverage Matters

Test coverage is important for several reasons:

1. **Identifies untested code**: It helps you identify parts of your code that haven't been tested.
2. **Guides test development**: It provides guidance on where to focus your testing efforts.
3. **Measures progress**: It allows you to track your progress in improving test quality.
4. **Builds confidence**: Higher coverage gives you more confidence in your code's correctness.
5. **Prevents regression**: Well-covered code is less likely to suffer from regression issues.

## Measuring Test Coverage in .NET

.NET provides several tools for measuring test coverage. In this chapter, we'll use the built-in coverage tools in the .NET CLI.

### Using the .NET CLI for Coverage

The .NET CLI includes a `coverage` command that can collect and report code coverage information. Here's how to use it:

```bash
cd TestTubes.Tests
dotnet test --collect:"XPlat Code Coverage"
```

This command runs your tests and collects coverage information. The coverage data is saved in a `.coverage` file in the `TestResults` directory.

To generate a human-readable report from the coverage data, you can use the `reportgenerator` tool:

```bash
dotnet tool install -g dotnet-reportgenerator-globaltool
reportgenerator -reports:"./TestResults/**/coverage.cobertura.xml" -targetdir:"./CoverageReport" -reporttypes:Html
```

This generates an HTML report in the `CoverageReport` directory. You can open `index.html` in a web browser to view the report.

## Analyzing Coverage Reports

The coverage report shows you:

1. **Overall coverage**: The percentage of code that's covered by tests.
2. **File-level coverage**: Coverage for each file in your project.
3. **Line-level coverage**: Which lines are covered and which aren't.
4. **Branch coverage**: Whether all branches of conditional statements are tested.

## Identifying Areas for Improvement

Based on the coverage report, we can identify areas that need more testing:

1. **Uncovered methods**: Methods that aren't executed by any test.
2. **Partially covered methods**: Methods where some branches or lines aren't executed.
3. **Error handling**: Exception handling code that isn't tested.
4. **Edge cases**: Code that handles edge cases or unusual inputs.

For example, in the TestTube project, we might find that:
- The `DeleteScientist` method in `ScientistController.cs` isn't fully covered
- The error handling in `UpdateEquipment` isn't tested
- The validation logic in `CreateScientist` isn't fully exercised

## Strategies for Improving Coverage

Once we've identified areas that need more testing, we can implement strategies to improve coverage:

1. **Write more tests**: Add tests for uncovered methods and edge cases.
2. **Test error conditions**: Add tests that trigger exceptions and error handling.
3. **Test boundary conditions**: Add tests for boundary values and edge cases.
4. **Refactor complex methods**: Break down complex methods into smaller, more testable methods.
5. **Use parameterized tests**: Use xUnit's `[Theory]` attribute to test multiple inputs with a single test method.

Let's implement one of these strategies for the TestTube project.

### Testing Validation Logic

Let's add a test for the validation logic in the for the POST operation to create a new scientist, but with some bad data.

```csharp
[Fact]
public async Task CreateScientist_WithInvalidData_ReturnsBadRequest()
{
    // Arrange
    var client = _testHelper.Client;
    var invalidScientist = new Scientist
    {
        Name = "", // Empty name should be invalid
        Specialty = "Test Specialty"
    };

    // Act
    var response = await client.PostAsJsonAsync("/scientist", invalidScientist);

    // Assert
    Assert.Equal(HttpStatusCode.BadRequest, response.StatusCode);
}
```

## Running Tests with Coverage

After adding these new tests, let's run the tests with coverage again to see if we've improved our coverage:

```bash
cd TestTubes.Tests
dotnet test --collect:"XPlat Code Coverage"
reportgenerator -reports:"./TestResults/**/coverage.cobertura.xml" -targetdir:"./CoverageReport" -reporttypes:Html
```

Check the updated coverage report to see if we've improved our coverage.

## Setting Coverage Goals

It's a good practice to set coverage goals for your project. For example, you might aim for:

- 80% overall coverage
- 90% coverage for critical components
- 100% coverage for core business logic

These goals can help guide your testing efforts and ensure that you're focusing on the most important parts of your code.

## Conclusion

In this chapter, we've learned how to measure and improve test coverage in a .NET project. We've seen how to use the .NET CLI to collect coverage data, how to analyze coverage reports to identify areas for improvement, and how to implement strategies to improve coverage.

Remember that while high coverage is a good goal, the quality of your tests is even more important. Focus on writing meaningful tests that verify the correct behavior of your code, not just tests that increase coverage.

Congratulations on completing the TestTube integration testing series! You now have the knowledge and skills to write effective integration tests for ASP.NET Core APIs with Entity Framework.

## Next Steps

You can also apply what you've learned to your own projects, writing integration tests for your APIs to ensure they work correctly.

[Return to Book 2 Table of Contents](../README.md)