# Testing Student Endpoints

## Learning Objectives
- Implement a basic controller for the Student resource
- Write focused integration tests for critical endpoints
- Test the most important operations (GET and POST)
- Learn how to make assertions about API responses

## Implementing a Simplified Student Controller

Let's create a controller with just the essential endpoints:

```csharp
// StudentExamTracker.API/Controllers/StudentsController.cs
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using StudentExamTracker.API.Data;
using StudentExamTracker.API.DTOs;
using StudentExamTracker.API.Models;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace StudentExamTracker.API.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class StudentsController : ControllerBase
    {
        private readonly ApplicationDbContext _context;

        public StudentsController(ApplicationDbContext context)
        {
            _context = context;
        }

        // GET: api/Students
        [HttpGet]
        public async Task<ActionResult<IEnumerable<StudentDto>>> GetStudents()
        {
            return await _context.Students
                .Select(s => new StudentDto
                {
                    Id = s.Id,
                    FirstName = s.FirstName,
                    LastName = s.LastName,
                    Email = s.Email,
                    DateOfBirth = s.DateOfBirth
                })
                .ToListAsync();
        }

        // GET: api/Students/5
        [HttpGet("{id}")]
        public async Task<ActionResult<StudentDto>> GetStudent(int id)
        {
            var student = await _context.Students.FindAsync(id);

            if (student == null)
            {
                return NotFound();
            }

            return new StudentDto
            {
                Id = student.Id,
                FirstName = student.FirstName,
                LastName = student.LastName,
                Email = student.Email,
                DateOfBirth = student.DateOfBirth
            };
        }

        // POST: api/Students
        [HttpPost]
        public async Task<ActionResult<StudentDto>> CreateStudent(StudentCreateDto studentDto)
        {
            var student = new Student
            {
                FirstName = studentDto.FirstName,
                LastName = studentDto.LastName,
                Email = studentDto.Email,
                DateOfBirth = studentDto.DateOfBirth
            };

            _context.Students.Add(student);
            await _context.SaveChangesAsync();

            return CreatedAtAction(
                nameof(GetStudent),
                new { id = student.Id },
                new StudentDto
                {
                    Id = student.Id,
                    FirstName = student.FirstName,
                    LastName = student.LastName,
                    Email = student.Email,
                    DateOfBirth = student.DateOfBirth
                });
        }
    }
}
```

## Writing Focused Tests for Student Endpoints

Let's create a test class that focuses only on the most critical endpoints:

```csharp
// StudentExamTracker.Tests/StudentControllerTests.cs
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
    public class StudentControllerTests : IntegrationTestBase
    {
        public StudentControllerTests(TestWebApplicationFactory<Program> factory) : base(factory)
        {
        }

        [Fact]
        public async Task GetStudents_ReturnsAllStudents()
        {
            // Act
            var students = await GetAsync<List<StudentDto>>("/api/students");

            // Assert
            students.Should().NotBeNull();
            students.Should().HaveCountGreaterThanOrEqualTo(2); // We seeded at least 2 students
            students.Should().Contain(s => s.Email == "test.student1@example.com");
            students.Should().Contain(s => s.Email == "test.student2@example.com");
        }

        [Fact]
        public async Task GetStudent_WithValidId_ReturnsStudent()
        {
            // Arrange
            var students = await GetAsync<List<StudentDto>>("/api/students");
            var studentId = students.Find(s => s.Email == "test.student1@example.com").Id;

            // Act
            var student = await GetAsync<StudentDto>($"/api/students/{studentId}");

            // Assert
            student.Should().NotBeNull();
            student.Email.Should().Be("test.student1@example.com");
            student.FirstName.Should().Be("Test");
            student.LastName.Should().Be("Student1");
        }

        [Fact]
        public async Task GetStudent_WithInvalidId_ReturnsNotFound()
        {
            // Act
            var response = await _client.GetAsync("/api/students/999");

            // Assert
            response.StatusCode.Should().Be(HttpStatusCode.NotFound);
        }

        [Fact]
        public async Task CreateStudent_WithValidData_ReturnsCreatedStudent()
        {
            // Arrange
            var newStudent = new StudentCreateDto
            {
                FirstName = "New",
                LastName = "Student",
                Email = "new.student@example.com",
                DateOfBirth = new DateTime(2002, 3, 3)
            };

            // Act
            var response = await PostAsync("/api/students", newStudent);
            var createdStudent = await response.Content.ReadFromJsonAsync<StudentDto>();

            // Assert
            response.StatusCode.Should().Be(HttpStatusCode.Created);
            createdStudent.Should().NotBeNull();
            createdStudent.Email.Should().Be("new.student@example.com");
            createdStudent.FirstName.Should().Be("New");
            createdStudent.LastName.Should().Be("Student");
            createdStudent.Id.Should().BeGreaterThan(0);

            // Verify the student was actually created
            var getResponse = await _client.GetAsync($"/api/students/{createdStudent.Id}");
            getResponse.StatusCode.Should().Be(HttpStatusCode.OK);
        }

        [Fact]
        public async Task CreateStudent_WithInvalidData_ReturnsBadRequest()
        {
            // Arrange
            var invalidStudent = new StudentCreateDto
            {
                // Missing required FirstName
                LastName = "Student",
                Email = "invalid.student@example.com",
                DateOfBirth = new DateTime(2002, 3, 3)
            };

            // Act
            var response = await PostAsync("/api/students", invalidStudent);

            // Assert
            response.StatusCode.Should().Be(HttpStatusCode.BadRequest);
        }
    }
}
```

## Understanding the Essential Tests

Let's examine the focused tests we've written:

### GET Tests

1. **GetStudents_ReturnsAllStudents**: Tests that the GET `/api/students` endpoint returns all students, including our seeded test data.

2. **GetStudent_WithValidId_ReturnsStudent**: Tests that the GET `/api/students/{id}` endpoint returns the correct student when given a valid ID.

3. **GetStudent_WithInvalidId_ReturnsNotFound**: Tests that the GET `/api/students/{id}` endpoint returns a 404 Not Found response when given an invalid ID.

### POST Tests

4. **CreateStudent_WithValidData_ReturnsCreatedStudent**: Tests that the POST `/api/students` endpoint creates a new student when given valid data and returns the created student with a 201 Created status code.

5. **CreateStudent_WithInvalidData_ReturnsBadRequest**: Tests that the POST `/api/students` endpoint returns a 400 Bad Request status code when given invalid data.

## Test Patterns and Best Practices

Our tests follow these key patterns:

1. **Arrange-Act-Assert**: Each test has clear sections:
   - **Arrange**: Set up test data
   - **Act**: Call the API endpoint
   - **Assert**: Verify the results

2. **Descriptive Test Names**: Names clearly describe what's being tested.

3. **Independent Tests**: Each test stands alone and doesn't depend on other tests.

4. **Verification**: For POST operations, we verify the data was actually created.

5. **Basic Validation**: We test both valid and invalid scenarios.

## Running the Tests

To run the tests:

```bash
cd StudentExamTracker.Tests
dotnet test
```

You should see output showing that all tests have passed.

## Next Steps

Now that we've tested the most critical Student endpoints, we can apply the same approach to the Exam endpoints if needed. However, the Student tests already demonstrate the core principles of integration testing.

By focusing on the most important endpoints, we've created a solid foundation for testing without overwhelming beginners with too many test cases.

[Next: Testing Exam Endpoints](./testy-exam-tests.md)