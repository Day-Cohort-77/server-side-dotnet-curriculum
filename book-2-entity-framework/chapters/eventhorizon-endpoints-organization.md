# Organizing Endpoints

## Feature Description

In this chapter, you'll set up the structure for organizing your API endpoints. A well-organized API is easier to maintain and extend as your application grows.

## Requirements

- Create endpoint organization files in the Endpoints directory:
  - VenueEndpoints.cs
  - EventEndpoints.cs
  - CategoryEndpoints.cs
  - RegistrationEndpoints.cs
  - AuthEndpoints.cs (for later use)

- Set up the basic structure for each endpoint file:
  - A static class with extension methods for the WebApplication
  - Methods to map endpoints for each resource
  - Organization by resource and operation type

- Configure Program.cs to:
  - Register the DbContext with the dependency injection container
  - Configure ASP.NET Core Identity:
    - Add Identity services with UserProfile and IdentityRole
    - Configure Identity options (password requirements, lockout, etc.)
    - Configure cookie authentication
  - Add authentication and authorization services
  - Set up CORS if needed
  - Configure the middleware pipeline:
    - UseAuthentication() and UseAuthorization() middleware
    - Call the endpoint mapping extension methods
  - Configure any other middleware required for the application

- No actual endpoint implementations yet, just the organizational structure

## Reflection

- Think about how to organize your endpoints to make the code maintainable.
- Consider grouping endpoints by resource type and using consistent naming conventions. This structure will make it easier to add new endpoints as you develop more features.
- For the Identity configuration, consider what password requirements, lockout policies, and cookie settings would be appropriate for your application.
- Remember that the order of middleware registration in Program.cs is important - authentication and authorization middleware should be registered before endpoint mapping.

Up Next: [DTOs and AutoMapper](./eventhorizon-dtos-automapper.md)