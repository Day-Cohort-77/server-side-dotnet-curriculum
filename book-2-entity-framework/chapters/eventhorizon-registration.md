# Registration Management

## Feature Description

In this chapter, you'll implement registration functionality for events. This will allow users to register for events, view their registrations, and cancel registrations.

## Requirements

- Implement the following endpoints in RegistrationEndpoints.cs:
  - POST /events/{id}/register - Registers current user for an event
  - GET /events/{id}/registrations - Returns all registrations for an event (admin only)
  - GET /users/registrations - Returns current user's registrations
  - DELETE /registrations/{id} - Cancels a registration

- For the POST /events/{id}/register endpoint:
  - Require authentication
  - Accept an event ID as a route parameter
  - Accept a JSON object in the request body with registration details:
    - AttendeeCount (number of people attending)
    - Notes (optional)
  - Validate the input data:
    - Event exists and is active
    - Event date is in the future
    - Event has not reached maximum capacity
    - AttendeeCount is positive and reasonable
  - Create a new Registration entity
  - Save the registration to the database
  - Return a 201 Created status code with the created registration
  - Return appropriate error messages for validation failures

- For the GET /events/{id}/registrations endpoint:
  - Require admin role
  - Accept an event ID as a route parameter
  - Return all registrations for the specified event
  - Include user information with each registration
  - Map the Registration entities to RegistrationDto objects using AutoMapper
  - Return a 200 OK status code with the list of registrations
  - Return a 404 Not Found status code if the event is not found

- For the GET /users/registrations endpoint:
  - Require authentication
  - Return all registrations for the current user
  - Include event information with each registration
  - Map the Registration entities to RegistrationDto objects using AutoMapper
  - Return a 200 OK status code with the list of registrations

- For the DELETE /registrations/{id} endpoint:
  - Require authentication
  - Accept a registration ID as a route parameter
  - Verify the registration belongs to the current user or user is admin
  - Delete the registration
  - Return a 204 No Content status code on successful deletion
  - Return a 404 Not Found status code if the registration is not found
  - Return a 403 Forbidden status code if not authorized

## Inputs and Outputs

**Inputs:**
- POST /events/{id}/register - Event ID in route, registration details in request body
- GET /events/{id}/registrations - Event ID in route, admin authentication
- GET /users/registrations - User authentication
- DELETE /registrations/{id} - Registration ID in route, user authentication

**Outputs:**
- POST /events/{id}/register - 201 Created with registration data
- GET /events/{id}/registrations - 200 OK with registrations array
- GET /users/registrations - 200 OK with registrations array
- DELETE /registrations/{id} - 204 No Content

## Reflection

Consider how to handle registration limits and prevent overbooking. When a user attempts to register for an event, you should check if adding their attendee count would exceed the event's maximum capacity. Think about what should happen if a user tries to register for an event that has already occurred or is at capacity.

Up Next: [Advanced Queries](./eventhorizon-advanced-queries.md)