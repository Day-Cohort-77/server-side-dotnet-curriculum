# User Roles and Authorization

## Feature Description

In this chapter, you'll implement role-based authorization for the API. This will allow you to restrict access to certain endpoints based on user roles.

## Requirements

- Set up role-based authorization:
  - Configure authorization policies in Program.cs
  - Create two default roles: "Admin" and "User"
  - Seed an admin user in the database

- Implement the following role management endpoints:
  - GET /roles - Returns all roles (admin only)
  - POST /roles - Creates a new role (admin only)
  - POST /users/{id}/roles - Assigns a role to a user (admin only)
  - DELETE /users/{id}/roles/{roleName} - Removes a role from a user (admin only)

- Apply authorization to existing endpoints:
  - Venue management endpoints (create, update, delete) - Admin only
  - Category management endpoints (create, update, delete) - Admin only
  - Event management endpoints:
    - Create/read - All authenticated users
    - Update/delete - Admin or event creator only
  - User profile - Users can only access their own profile

- For the GET /roles endpoint:
  - Require admin role
  - Return a list of all roles
  - Return a 200 OK status code with the list of roles
  - Return a 403 Forbidden status code if not an admin

- For the POST /roles endpoint:
  - Require admin role
  - Accept a JSON object in the request body with role details
  - Create a new role
  - Return a 201 Created status code with the created role
  - Return a 403 Forbidden status code if not an admin

- For the POST /users/{id}/roles endpoint:
  - Require admin role
  - Accept a user ID as a route parameter
  - Accept a JSON object in the request body with role name
  - Assign the role to the user
  - Return a 200 OK status code on success
  - Return appropriate error codes for failures

- For the DELETE /users/{id}/roles/{roleName} endpoint:
  - Require admin role
  - Accept a user ID and role name as route parameters
  - Remove the role from the user
  - Return a 204 No Content status code on success
  - Return appropriate error codes for failures

## Inputs and Outputs

**Inputs:**
- GET /roles - Admin authentication
- POST /roles - Admin authentication, role data in request body
- POST /users/{id}/roles - Admin authentication, user ID in route, role name in request body
- DELETE /users/{id}/roles/{roleName} - Admin authentication, user ID and role name in route

**Outputs:**
- GET /roles - 200 OK with roles array
- POST /roles - 201 Created with role data
- POST /users/{id}/roles - 200 OK with confirmation
- DELETE /users/{id}/roles/{roleName} - 204 No Content

## Reflection

Consider how to handle authorization failures gracefully. Return appropriate status codes (401 Unauthorized for unauthenticated requests, 403 Forbidden for authenticated but unauthorized requests) and clear error messages. Think about how to implement the "event creator can edit their own events" policy, which requires more complex authorization logic than simple role checks.

Up Next: [Registration Management](./eventhorizon-registration.md)