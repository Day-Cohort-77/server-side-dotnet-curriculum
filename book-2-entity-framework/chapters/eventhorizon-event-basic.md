# Event Endpoints - Basic Operations

## Feature Description

In this chapter, you'll implement basic endpoints for event management. These endpoints will allow users to view and create events.

## Requirements

- Implement the following endpoints in EventEndpoints.cs:
  - GET /events - Returns all events (with optional filtering by date)
  - GET /events/{id} - Returns a specific event with venue and category details
  - POST /events - Creates a new event

- For the GET /events endpoint:
  - Return a list of all active events
  - Include optional query parameters for filtering:
    - fromDate - Filter events starting on or after this date
    - toDate - Filter events starting on or before this date
    - categoryId - Filter events by category
  - Map the Event entities to EventDto objects using AutoMapper
  - Include basic venue and category information in the response
  - Return a 200 OK status code with the list of events
  - Consider pagination for large result sets

- For the GET /events/{id} endpoint:
  - Accept an event ID as a route parameter
  - Return the event with the specified ID
  - Include detailed venue and category information
  - Map the Event entity to an EventDetailDto using AutoMapper
  - Return a 200 OK status code if the event is found
  - Return a 404 Not Found status code if the event is not found

- For the POST /events endpoint:
  - Accept a JSON object in the request body with event details
  - Validate the input data, including:
    - Required fields are provided
    - Date is in the future
    - MaxAttendees is within reasonable limits
    - VenueId and EventCategoryId reference existing entities
  - Create a new Event entity
  - Save the event to the database
  - Return a 201 Created status code with the created event
  - Include the Location header with the URL to the new resource

## Inputs and Outputs

**Inputs:**
- GET /events - Optional query parameters (fromDate, toDate, categoryId)
- GET /events/{id} - Event ID as a route parameter
- POST /events - Event data in request body

**Outputs:**
- GET /events - A JSON array of event objects
- GET /events/{id} - A JSON object representing a single event with details
- POST /events - 201 Created with event data

## Reflection

Consider what date format would be most appropriate for the API. ISO 8601 (e.g., "2025-07-15T19:00:00Z") is a good standard for dates in APIs. Think about validation rules for events, such as ensuring the event date is in the future and that the venue exists and has sufficient capacity.

Up Next: [Event Endpoints - Advanced Operations](./eventhorizon-event-advanced.md)