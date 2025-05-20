# Understanding Entity Relationships in Testing

## Learning Objectives
- Understand how to test endpoints with related entities
- Learn which relationship tests are most important
- Apply our focused testing approach to related entities

## The Challenge of Testing Relationships

When testing endpoints that involve relationships between entities (like StudentExam, which connects Students and Exams), we face additional challenges:

1. **Foreign key validation**: Ensuring that related entities exist
2. **Data integrity**: Maintaining proper relationships between entities
3. **Related data loading**: Verifying that related data is properly included in responses

Rather than implementing a full suite of tests for these relationships, we can apply our focused approach to test the most critical aspects.

## Key Relationship Tests

When testing relationships, focus on these key aspects:

1. **Creating related entities**: Test that you can create a StudentExam that connects a Student and an Exam
2. **Retrieving related data**: Test that when you get a StudentExam, it includes information about the related Student and Exam
3. **Foreign key validation**: Test that you can't create a StudentExam with invalid Student or Exam IDs

## Example Test Structure

Here's how you would structure tests for the StudentExam endpoints, focusing on the most critical relationship aspects:

```csharp
// StudentExamTracker.Tests/StudentExamControllerTests.cs
using FluentAssertions;
using StudentExamTracker.API.DTOs;
using StudentExamTracker.Tests.Helpers;
using System;
using System.Collections.Generic;
using System.Net;
using System.Net.Http.Json;
using System.Threading.Tasks;
using Xunit;

namespace StudentExamTracker.Tests
{
    public class StudentExamControllerTests : IntegrationTestBase
    {
        public StudentExamControllerTests(TestWebApplicationFactory<Program> factory) : base(factory)
        {
        }

        [Fact]
        public async Task GetStudentExams_ReturnsAllStudentExams()
        {
            // Act
            var studentExams = await GetAsync<List<StudentExamDto>>("/api/studentexams");

            // Assert
            studentExams.Should().NotBeNull();
            studentExams.Should().HaveCountGreaterThanOrEqualTo(2);
        }

        [Fact]
        public async Task GetStudentExam_WithValidId_IncludesRelatedData()
        {
            // Arrange
            var studentExams = await GetAsync<List<StudentExamDto>>("/api/studentexams");
            var studentExamId = studentExams[0].Id;

            // Act
            var studentExam = await GetAsync<StudentExamDto>($"/api/studentexams/{studentExamId}");

            // Assert
            studentExam.Should().NotBeNull();
            studentExam.StudentName.Should().NotBeNullOrEmpty();
            studentExam.ExamName.Should().NotBeNullOrEmpty();
        }

        [Fact]
        public async Task CreateStudentExam_WithValidData_ReturnsCreatedStudentExam()
        {
            // Arrange
            var students = await GetAsync<List<StudentDto>>("/api/students");
            var exams = await GetAsync<List<ExamDto>>("/api/exams");

            var newStudentExam = new StudentExamCreateDto
            {
                StudentId = students[0].Id,
                ExamId = exams[0].Id,
                Score = 95,
                DateTaken = DateTime.Now.Date
            };

            // Act
            var response = await PostAsync("/api/studentexams", newStudentExam);
            var createdStudentExam = await response.Content.ReadFromJsonAsync<StudentExamDto>();

            // Assert
            response.StatusCode.Should().Be(HttpStatusCode.Created);
            createdStudentExam.Should().NotBeNull();
            createdStudentExam.StudentId.Should().Be(newStudentExam.StudentId);
            createdStudentExam.ExamId.Should().Be(newStudentExam.ExamId);
            createdStudentExam.Score.Should().Be(newStudentExam.Score);
        }

        [Fact]
        public async Task CreateStudentExam_WithInvalidStudentId_ReturnsBadRequest()
        {
            // Arrange
            var exams = await GetAsync<List<ExamDto>>("/api/exams");

            var invalidStudentExam = new StudentExamCreateDto
            {
                StudentId = 999, // Invalid student ID
                ExamId = exams[0].Id,
                Score = 95,
                DateTaken = DateTime.Now.Date
            };

            // Act
            var response = await PostAsync("/api/studentexams", invalidStudentExam);

            // Assert
            response.StatusCode.Should().Be(HttpStatusCode.BadRequest);
        }
    }
}
```

## Simplifying Relationship Testing

When testing relationships, remember these principles:

1. **Focus on creation and retrieval**: These are the most critical operations
2. **Verify key relationships**: Ensure that related entities are properly connected
3. **Test basic validation**: Verify that you can't create invalid relationships
4. **Keep tests simple**: Don't try to test every possible scenario

## Moving Forward

By applying our focused testing approach to relationships, we can ensure that the most critical aspects of our API work correctly without getting overwhelmed by too many test cases.

Remember that the goal is to build confidence in your API's functionality, not to test every possible edge case. By focusing on the most important operations and relationships, you'll create a solid foundation for your application.

## Conclusion

Testing relationships between entities is an important part of integration testing, but it doesn't have to be overwhelming. By focusing on the most critical aspects of relationships, you can create effective tests that give you confidence in your API's functionality.

In the next chapter, we'll discuss the value of focused testing and how to apply these principles to your own projects.

[Next: Achieving 100% Test Coverage](./testy-coverage.md)