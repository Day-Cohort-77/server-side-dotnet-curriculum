# Venue Endpoints - Read Operations

## Feature Description

In this chapter, you'll implement endpoints to retrieve venue information. These will be your first functional API endpoints in the EventHorizon project.

## Requirements

- Implement the following endpoints in VenueEndpoints.cs:
  - GET /venues - Returns all venues
  - GET /venues/{id} - Returns a specific venue by ID

- For the GET /venues endpoint:
  - Return a list of all active venues
  - Map the Venue entities to VenueDto objects using AutoMapper
  - Return a 200 OK status code with the list of venues
  - Consider adding optional filtering (e.g., by name or capacity)

- For the GET /venues/{id} endpoint:
  - Accept a venue ID as a route parameter
  - Return the venue with the specified ID
  - Map the Venue entity to a VenueDto using AutoMapper
  - Return a 200 OK status code if the venue is found
  - Return a 404 Not Found status code if the venue is not found

- Responses should include venue details but not related events

## Inputs and Outputs

**Inputs:**
- GET /venues - No required inputs
- GET /venues/{id} - Venue ID as a route parameter

**Outputs:**
- GET /venues - A JSON array of venue objects
- GET /venues/{id} - A JSON object representing a single venue

## Reflection

Consider what information would be most useful in the response. Think about error handling for cases where a venue doesn't exist. How will you handle the case where a venue exists but is marked as inactive?

Up Next: [Venue Endpoints - Write Operations](./eventhorizon-venue-write.md)