# Setting Up the Database Context

## Learning Objectives
- Create a simple DbContext class for our application
- Configure basic entity relationships
- Implement minimal database seeding for testing
- Understand why test data is important

## Creating a Simple DbContext

The `DbContext` class connects our models to the database. Let's create a streamlined version:

```bash
mkdir -p StudentExamTracker.API/Data
```

Now, let's create our `ApplicationDbContext` class:

```csharp
// StudentExamTracker.API/Data/ApplicationDbContext.cs
using Microsoft.EntityFrameworkCore;
using StudentExamTracker.API.Models;
using System;

namespace StudentExamTracker.API.Data
{
    public class ApplicationDbContext : DbContext
    {
        public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
            : base(options)
        {
        }

        public DbSet<Student> Students { get; set; }
        public DbSet<Exam> Exams { get; set; }
        public DbSet<StudentExam> StudentExams { get; set; }

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            base.OnModelCreating(modelBuilder);

            // Configure basic relationships
            modelBuilder.Entity<StudentExam>()
                .HasOne(se => se.Student)
                .WithMany(s => s.StudentExams)
                .HasForeignKey(se => se.StudentId);

            modelBuilder.Entity<StudentExam>()
                .HasOne(se => se.Exam)
                .WithMany(e => e.StudentExams)
                .HasForeignKey(se => se.ExamId);

            // Add seed data
            SeedData(modelBuilder);
        }

        private void SeedData(ModelBuilder modelBuilder)
        {
            // Seed minimal test data

            // Two students
            modelBuilder.Entity<Student>().HasData(
                new Student
                {
                    Id = 1,
                    FirstName = "John",
                    LastName = "Doe",
                    Email = "john.doe@example.com",
                    DateOfBirth = new DateTime(2000, 1, 15)
                },
                new Student
                {
                    Id = 2,
                    FirstName = "Jane",
                    LastName = "Smith",
                    Email = "jane.smith@example.com",
                    DateOfBirth = new DateTime(2001, 5, 20)
                }
            );

            // Two exams
            modelBuilder.Entity<Exam>().HasData(
                new Exam
                {
                    Id = 1,
                    Name = "Introduction to C#",
                    Subject = "Programming",
                    NumberOfQuestions = 50
                },
                new Exam
                {
                    Id = 2,
                    Name = "Database Fundamentals",
                    Subject = "Databases",
                    NumberOfQuestions = 40
                }
            );

            // Two student exams
            modelBuilder.Entity<StudentExam>().HasData(
                new StudentExam
                {
                    Id = 1,
                    StudentId = 1,
                    ExamId = 1,
                    Score = 85,
                    DateTaken = DateTime.Now.AddDays(-10)
                },
                new StudentExam
                {
                    Id = 2,
                    StudentId = 2,
                    ExamId = 2,
                    Score = 92,
                    DateTaken = DateTime.Now.AddDays(-5)
                }
            );
        }
    }
}
```

## Registering the DbContext in Program.cs

Let's update our `Program.cs` file to register the DbContext:

```csharp
// StudentExamTracker.API/Program.cs
using Microsoft.EntityFrameworkCore;
using StudentExamTracker.API.Data;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddControllers();

// Add DbContext with in-memory database for simplicity
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseInMemoryDatabase("StudentExamDb"));

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();

// Make the Program class public for testing
public partial class Program { }
```

## Simple Test Data Seeder

For our tests, we need a basic way to seed the database with test data:

```csharp
// StudentExamTracker.Tests/Helpers/TestDataSeeder.cs
using StudentExamTracker.API.Data;
using StudentExamTracker.API.Models;
using System;

namespace StudentExamTracker.Tests.Helpers
{
    public static class TestDataSeeder
    {
        public static void InitializeDbForTests(ApplicationDbContext db)
        {
            // Clear any existing data
            db.StudentExams.RemoveRange(db.StudentExams);
            db.Students.RemoveRange(db.Students);
            db.Exams.RemoveRange(db.Exams);
            db.SaveChanges();

            // Add test students
            var student1 = new Student
            {
                FirstName = "Test",
                LastName = "Student1",
                Email = "test.student1@example.com",
                DateOfBirth = new DateTime(2000, 1, 1)
            };

            var student2 = new Student
            {
                FirstName = "Test",
                LastName = "Student2",
                Email = "test.student2@example.com",
                DateOfBirth = new DateTime(2001, 2, 2)
            };

            db.Students.AddRange(student1, student2);
            db.SaveChanges();

            // Add test exams
            var exam1 = new Exam
            {
                Name = "Test Exam 1",
                Subject = "Test Subject 1",
                NumberOfQuestions = 30
            };

            var exam2 = new Exam
            {
                Name = "Test Exam 2",
                Subject = "Test Subject 2",
                NumberOfQuestions = 25
            };

            db.Exams.AddRange(exam1, exam2);
            db.SaveChanges();

            // Add test student exams
            db.StudentExams.AddRange(
                new StudentExam
                {
                    StudentId = student1.Id,
                    ExamId = exam1.Id,
                    Score = 90,
                    DateTaken = DateTime.Now.AddDays(-1)
                },
                new StudentExam
                {
                    StudentId = student2.Id,
                    ExamId = exam2.Id,
                    Score = 85,
                    DateTaken = DateTime.Now.AddDays(-2)
                }
            );

            db.SaveChanges();
        }
    }
}
```

## Updating the Test Factory

Let's update our `TestWebApplicationFactory` to use the seeder:

```csharp
// StudentExamTracker.Tests/Helpers/TestWebApplicationFactory.cs
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Mvc.Testing;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.DependencyInjection;
using StudentExamTracker.API.Data;
using System.Linq;

namespace StudentExamTracker.Tests.Helpers
{
    public class TestWebApplicationFactory<TProgram>
        : WebApplicationFactory<TProgram> where TProgram : class
    {
        protected override void ConfigureWebHost(IWebHostBuilder builder)
        {
            builder.ConfigureServices(services =>
            {
                // Replace the real database with in-memory database
                var descriptor = services.SingleOrDefault(
                    d => d.ServiceType == typeof(DbContextOptions<ApplicationDbContext>));

                if (descriptor != null)
                {
                    services.Remove(descriptor);
                }

                services.AddDbContext<ApplicationDbContext>(options =>
                {
                    options.UseInMemoryDatabase("TestingDb");
                });

                // Seed the test database
                var sp = services.BuildServiceProvider();
                using (var scope = sp.CreateScope())
                {
                    var db = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
                    db.Database.EnsureCreated();
                    TestDataSeeder.InitializeDbForTests(db);
                }
            });
        }
    }
}
```

## Why Test Data Matters

Database seeding is important for integration testing because:

1. **Consistency**: Tests can rely on known data being in the database
2. **Isolation**: Each test runs against a fresh database state
3. **Readability**: You can reference known entities by their properties

Our approach uses:

1. **Application Seeding**: Basic seed data in the `ApplicationDbContext` for the real application
2. **Test Seeding**: Specific test data in the `TestDataSeeder` for our tests

## Next Steps

In the next chapter, we'll implement the Student endpoints in our API and write focused integration tests for the most critical operations (GET and POST).

With our models, DbContext, and test infrastructure in place, we're ready to start building and testing the essential parts of our API!

[Next: Testing Student Endpoints](./testy-student-tests.md)