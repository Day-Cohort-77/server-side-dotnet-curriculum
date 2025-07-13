# User Authentication

## Feature Description

In this chapter, you'll implement user authentication for the API. This will allow users to register, log in, and access their profile information.

## Requirements

- Set up ASP.NET Core Identity:
  - Configure Identity in Program.cs
  - Create a User model that extends IdentityUser
  - Update the DbContext to inherit from IdentityDbContext

- Implement the following endpoints in AuthEndpoints.cs:
  - POST /auth/register - Registers a new user
  - POST /auth/login - Authenticates a user and returns a token/cookie
  - GET /auth/me - Returns the current user's profile

- For the POST /auth/register endpoint:
  - Accept a JSON object in the request body with user registration details:
    - Username
    - Email
    - Password
    - FirstName
    - LastName
  - Validate the input data
  - Create a new user using ASP.NET Core Identity
  - Return a 201 Created status code with basic user information (excluding password)
  - Return appropriate error messages for validation failures

- For the POST /auth/login endpoint:
  - Accept a JSON object in the request body with login credentials:
    - Username or Email
    - Password
  - Validate the credentials using ASP.NET Core Identity
  - Generate an authentication token or set up a cookie-based authentication
  - Return a 200 OK status code with the token or confirmation of successful login
  - Return a 401 Unauthorized status code for invalid credentials

- For the GET /auth/me endpoint:
  - Require authentication
  - Return the current user's profile information
  - Map the User entity to a UserDto using AutoMapper
  - Return a 200 OK status code with the user profile
  - Return a 401 Unauthorized status code if not authenticated

## Inputs and Outputs

**Inputs:**
- POST /auth/register - User registration data in request body
- POST /auth/login - Login credentials in request body
- GET /auth/me - Authentication token in header or cookie

**Outputs:**
- POST /auth/register - 201 Created with user data
- POST /auth/login - 200 OK with token or confirmation
- GET /auth/me - 200 OK with user profile data

## Reflection

Consider what validation rules would be appropriate for user registration. Password requirements should be reasonably strong but not overly complex. Think about how to handle email verification if needed. For authentication tokens, consider using JWT (JSON Web Tokens) as they are widely supported and provide a stateless authentication mechanism.

Up Next: [User Roles and Authorization](./eventhorizon-authorization.md)