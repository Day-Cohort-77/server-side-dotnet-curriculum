# Creating the API Models

## Learning Objectives
- Create the essential domain models for our Student Exam Tracker API
- Implement basic data annotations for validation
- Create simple DTOs (Data Transfer Objects) for our API endpoints
- Understand the core model relationships

## Domain Models

Let's create the essential models for our Student Exam Tracker API. We'll focus on the three main entities: `Student`, `Exam`, and `StudentExam`.

First, create a Models directory:

```bash
mkdir -p StudentExamTracker.API/Models
```

### Student Model

The `Student` model contains basic information about students:

```csharp
// StudentExamTracker.API/Models/Student.cs
using System;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;

namespace StudentExamTracker.API.Models
{
    public class Student
    {
        public int Id { get; set; }

        [Required]
        [StringLength(50)]
        public string FirstName { get; set; }

        [Required]
        [StringLength(50)]
        public string LastName { get; set; }

        [Required]
        [EmailAddress]
        public string Email { get; set; }

        public DateTime DateOfBirth { get; set; }

        // Navigation property
        public List<StudentExam> StudentExams { get; set; }
    }
}
```

### Exam Model

The `Exam` model contains information about exams:

```csharp
// StudentExamTracker.API/Models/Exam.cs
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;

namespace StudentExamTracker.API.Models
{
    public class Exam
    {
        public int Id { get; set; }

        [Required]
        [StringLength(100)]
        public string Name { get; set; }

        [Required]
        public string Subject { get; set; }

        public int NumberOfQuestions { get; set; }

        // Navigation property
        public List<StudentExam> StudentExams { get; set; }
    }
}
```

### StudentExam Model

The `StudentExam` model links students to exams and stores their scores:

```csharp
// StudentExamTracker.API/Models/StudentExam.cs
using System;
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace StudentExamTracker.API.Models
{
    public class StudentExam
    {
        public int Id { get; set; }

        [Required]
        public int StudentId { get; set; }

        [Required]
        public int ExamId { get; set; }

        [Range(0, 100)]
        public int Score { get; set; }

        public DateTime DateTaken { get; set; }

        // Navigation properties
        [ForeignKey("StudentId")]
        public Student Student { get; set; }

        [ForeignKey("ExamId")]
        public Exam Exam { get; set; }
    }
}
```

## Data Transfer Objects (DTOs)

Let's create simple DTOs for our models. DTOs help us control what data is exposed through our API.

First, create a DTOs directory:

```bash
mkdir -p StudentExamTracker.API/DTOs
```

### Student DTOs

```csharp
// StudentExamTracker.API/DTOs/StudentDtos.cs
using System;
using System.ComponentModel.DataAnnotations;

namespace StudentExamTracker.API.DTOs
{
    // DTO for returning student data
    public class StudentDto
    {
        public int Id { get; set; }
        public string FirstName { get; set; }
        public string LastName { get; set; }
        public string Email { get; set; }
        public DateTime DateOfBirth { get; set; }
        public string FullName => $"{FirstName} {LastName}";
    }

    // DTO for creating a student
    public class StudentCreateDto
    {
        [Required]
        public string FirstName { get; set; }

        [Required]
        public string LastName { get; set; }

        [Required]
        [EmailAddress]
        public string Email { get; set; }

        public DateTime DateOfBirth { get; set; }
    }
}
```

### Exam DTOs

```csharp
// StudentExamTracker.API/DTOs/ExamDtos.cs
using System.ComponentModel.DataAnnotations;

namespace StudentExamTracker.API.DTOs
{
    // DTO for returning exam data
    public class ExamDto
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public string Subject { get; set; }
        public int NumberOfQuestions { get; set; }
    }

    // DTO for creating an exam
    public class ExamCreateDto
    {
        [Required]
        public string Name { get; set; }

        [Required]
        public string Subject { get; set; }

        public int NumberOfQuestions { get; set; }
    }
}
```

### StudentExam DTOs

```csharp
// StudentExamTracker.API/DTOs/StudentExamDtos.cs
using System;
using System.ComponentModel.DataAnnotations;

namespace StudentExamTracker.API.DTOs
{
    // DTO for returning student exam data
    public class StudentExamDto
    {
        public int Id { get; set; }
        public int StudentId { get; set; }
        public int ExamId { get; set; }
        public int Score { get; set; }
        public DateTime DateTaken { get; set; }

        // Include related data
        public string StudentName { get; set; }
        public string ExamName { get; set; }
    }

    // DTO for creating a student exam
    public class StudentExamCreateDto
    {
        [Required]
        public int StudentId { get; set; }

        [Required]
        public int ExamId { get; set; }

        [Range(0, 100)]
        public int Score { get; set; }

        public DateTime DateTaken { get; set; }
    }
}
```

## Understanding Core Model Relationships

Our models form a simple many-to-many relationship:

- A student can take many exams
- An exam can be taken by many students
- The `StudentExam` model serves as a join table with the score and date

This is a common pattern in database design that Entity Framework Core handles well.

## Basic Data Annotations

We've used a few essential data annotations:

- `[Required]`: Indicates that a property must have a value
- `[Range]`: Specifies minimum and maximum values for numeric types
- `[EmailAddress]`: Ensures that a string is a valid email address
- `[ForeignKey]`: Specifies a foreign key relationship

These annotations serve two purposes:
1. They configure how Entity Framework Core creates the database schema
2. They provide basic validation for incoming data

## Next Steps

In the next chapter, we'll create a simple `DbContext` class and set up basic database seeding. This will give us a consistent test environment for our integration tests.

After that, we'll implement the most important API endpoints and write focused tests for them.

[Next: Seeding Test Data](./testy-dbcontext.md)