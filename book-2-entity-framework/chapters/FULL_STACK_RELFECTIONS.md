# Full Stack Reflection Questions

Use these questions to assess your understanding and identify areas where you might need additional research or practice.

## Using These Questions for Self-Assessment

As you work through these reflection questions, consider the following approaches to maximize their benefit:

1. **Verbalize your answers**: Practice articulating your understanding out loud or in writing, as if you were explaining the concept to someone else or in a technical interview.

2. **Connect concepts**: Look for relationships between different topics. For example, how does your understanding of Entity Framework Core relate to API design or authentication?

3. **Apply to real projects**: Think about how you've implemented these concepts in your projects, or how you would implement them in future work.

4. **Identify knowledge gaps**: Be honest about areas where your understanding is incomplete. Use these as opportunities for focused learning.

5. **Revisit regularly**: Return to these questions periodically as you gain more experience. Your understanding will deepen over time.

6. **Discuss with peers**: Share your thoughts with classmates or colleagues. Different perspectives can enhance your understanding.

Remember that the goal isn't just to have "correct" answers, but to develop a nuanced understanding of how these technologies and concepts work together in real-world applications.

## C# and .NET

- Could you explain the benefit of providing an initial value for properties in a C# class? How does this relate to object initialization and constructor design?
- What are the benefits of using inheritance?
- Explain the concept of dependency injection in .NET. How does it improve testability and maintainability?

## Entity Framework Core

- How did navigation properties in Entity Framework Core help you work with related data in your projects? What advantages did they provide?
- Describe the role of DbContext in Entity Framework Core.
- How has working with Entity Framework Core changed your understanding of database interactions compared to writing raw SQL queries?

## Data Transfer Objects (DTOs)

- What problem do DTOs solve in a web API architecture? Provide specific examples of when DTOs are beneficial.
- How do DTOs differ from database models, and why is this separation important in an application?
- How do DTOs contribute to API security? Provide examples of security vulnerabilities they can help prevent.

## Authentication and Authorization

- Compare and contrast authentication and authorization in the context of web applications. How do they work together?
- Explain the authentication method used in the projects in this book.
- How would you implement role-based authorization in an ASP.NET Core API that supports the operation of a doctor's office? What considerations are important when designing roles?
- How did implementing authentication change your perspective on API security and user data protection?

## Testing in .NET Applications

- How did your experience with integration testing in the TestTube project change your understanding of API reliability?
- What challenges, if any, did you encounter when setting up tests for your API endpoints?
- How did the WebApplicationFactory help you test your API without needing a real HTTP server?
- What benefits did you discover from writing tests for your API endpoints?

## Full Stack Development Lifecycle

- After building several API projects, how has your understanding of the client-server relationship evolved?
- You are building an inventory system with a React-based client and an ASP.NET API. The user is viewing a list of products in inventory. Each product name is a hyperlink. Can you describe the technical processes that happen from when one of the links is clicked to when the details are rendered? Be as technically detailed as possible.
- Why would it be bad if the client connected directly to the database? What security and architectural issues would this create?
- How did organizing your code into separate concerns (models, endpoints, DTOs, etc...) help you manage the complexity of your applications?

## API Design and Organization

- Based on your experience with these projects, how does organizing endpoints by resource type improve the maintainability of your API?
- How did your understanding of database relationships (one-to-many, many-to-many) influence your API design decisions?
- What considerations did you find important when designing your data models and their relationships?
- How did separating your application into different components (models, DTOs, endpoints) help you manage complexity?

