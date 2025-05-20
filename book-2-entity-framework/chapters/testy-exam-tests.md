# Applying the Same Testing Approach

In the previous chapter, we learned how to test the Student endpoints by focusing on the most critical operations (GET and POST). The same principles can be applied to testing the Exam endpoints.

## Key Points to Remember

When testing the Exam endpoints, remember these key points:

1. **Focus on critical operations**: Prioritize testing GET and POST operations
2. **Keep tests simple**: Write clear, focused tests that are easy to understand
3. **Test both success and failure cases**: Verify that validation works correctly
4. **Use descriptive test names**: Make it clear what each test is checking

## Example Test Structure

Here's how you would structure tests for the Exam endpoints, following the same pattern we used for Student endpoints:

```csharp
// StudentExamTracker.Tests/ExamControllerTests.cs
using FluentAssertions;
using StudentExamTracker.API.DTOs;
using StudentExamTracker.Tests.Helpers;
using System.Collections.Generic;
using System.Net;
using System.Net.Http.Json;
using System.Threading.Tasks;
using Xunit;

namespace StudentExamTracker.Tests
{
    public class ExamControllerTests : IntegrationTestBase
    {
        public ExamControllerTests(TestWebApplicationFactory<Program> factory) : base(factory)
        {
        }

        [Fact]
        public async Task GetExams_ReturnsAllExams()
        {
            // Act
            var exams = await GetAsync<List<ExamDto>>("/api/exams");

            // Assert
            exams.Should().NotBeNull();
            exams.Should().HaveCountGreaterThanOrEqualTo(2);
            exams.Should().Contain(e => e.Name == "Test Exam 1");
        }

        [Fact]
        public async Task GetExam_WithValidId_ReturnsExam()
        {
            // Arrange
            var exams = await GetAsync<List<ExamDto>>("/api/exams");
            var examId = exams.Find(e => e.Name == "Test Exam 1").Id;

            // Act
            var exam = await GetAsync<ExamDto>($"/api/exams/{examId}");

            // Assert
            exam.Should().NotBeNull();
            exam.Name.Should().Be("Test Exam 1");
            exam.Subject.Should().Be("Test Subject 1");
        }

        [Fact]
        public async Task CreateExam_WithValidData_ReturnsCreatedExam()
        {
            // Arrange
            var newExam = new ExamCreateDto
            {
                Name = "New Exam",
                Subject = "New Subject",
                NumberOfQuestions = 40
            };

            // Act
            var response = await PostAsync("/api/exams", newExam);
            var createdExam = await response.Content.ReadFromJsonAsync<ExamDto>();

            // Assert
            response.StatusCode.Should().Be(HttpStatusCode.Created);
            createdExam.Should().NotBeNull();
            createdExam.Name.Should().Be("New Exam");
            createdExam.Subject.Should().Be("New Subject");
        }
    }
}
```

## Moving Forward

Rather than implementing all the tests for Exam endpoints, we recommend applying what you've learned from the Student endpoint tests. The patterns and principles are the same:

1. Create a controller with the essential endpoints (GET and POST)
2. Write focused tests for those endpoints
3. Verify both success and failure cases

By focusing on these core operations, you'll build a solid foundation for testing without getting overwhelmed by too many test cases.

## Next Steps

In a real-world application, you might also need to test relationships between entities. For example, the StudentExam entity connects Students and Exams. The same testing principles apply, but you would need to ensure that the relationships are correctly maintained.

Remember that the goal is not to test everything, but to test the most critical parts of your application in a way that gives you confidence that it works correctly.

[Next: Testing StudentExam Endpoints](./testy-studentexam-tests.md)