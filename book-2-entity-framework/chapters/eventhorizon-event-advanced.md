# Event Endpoints - Advanced Operations

## Feature Description

In this chapter, you'll implement advanced event management features. These endpoints will allow users to update, delete, and search for events.

## Requirements

- Implement the following endpoints in EventEndpoints.cs:
  - PUT /events/{id} - Updates an event
  - DELETE /events/{id} - Cancels/deletes an event
  - GET /events/category/{categoryId} - Returns events by category
  - GET /events/search?q={searchTerm} - Searches events by name or description

- For the PUT /events/{id} endpoint:
  - Accept an event ID as a route parameter
  - Accept a JSON object in the request body with updated event details
  - Validate the input data
  - Update the existing event
  - Return a 200 OK status code with the updated event
  - Return a 404 Not Found status code if the event is not found
  - Consider validation rules for updates (e.g., can't change date if registrations exist)

- For the DELETE /events/{id} endpoint:
  - Accept an event ID as a route parameter
  - Check if the event has any registrations
  - If registrations exist, consider options:
    - Option 1: Prevent deletion and return an error
    - Option 2: Soft delete by setting IsActive to false
    - Option 3: Delete the event and cascade delete registrations
  - Return a 204 No Content status code on successful deletion
  - Return a 404 Not Found status code if the event is not found

- For the GET /events/category/{categoryId} endpoint:
  - Accept a category ID as a route parameter
  - Return all active events in the specified category
  - Map the Event entities to EventDto objects using AutoMapper
  - Return a 200 OK status code with the list of events
  - Return a 404 Not Found status code if the category is not found

- For the GET /events/search?q={searchTerm} endpoint:
  - Accept a search term as a query parameter
  - Search for events where the name or description contains the search term
  - Return matching events
  - Map the Event entities to EventDto objects using AutoMapper
  - Return a 200 OK status code with the list of events
  - Return an empty array if no matches are found

## Inputs and Outputs

**Inputs:**
- PUT /events/{id} - Event ID in route, updated event data in request body
- DELETE /events/{id} - Event ID in route
- GET /events/category/{categoryId} - Category ID in route
- GET /events/search?q={searchTerm} - Search term in query parameter

**Outputs:**
- PUT /events/{id} - 200 OK with updated event data
- DELETE /events/{id} - 204 No Content or appropriate error
- GET /events/category/{categoryId} - A JSON array of event objects
- GET /events/search?q={searchTerm} - A JSON array of event objects

## Reflection

Think about what should happen to registrations when an event is canceled. Should you allow deletion and cascade to delete the registrations, or prevent deletion and return an error? Consider implementing a soft delete approach where events are marked as canceled rather than being removed from the database.

Up Next: [User Authentication](./eventhorizon-authentication.md)