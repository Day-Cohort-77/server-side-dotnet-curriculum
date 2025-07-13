# Venue Endpoints - Write Operations

## Feature Description

In this chapter, you'll implement endpoints to create, update, and delete venues. These endpoints will allow for complete management of venue resources.

## Requirements

- Implement the following endpoints in VenueEndpoints.cs:
  - POST /venues - Creates a new venue
  - PUT /venues/{id} - Updates an existing venue
  - DELETE /venues/{id} - Deletes a venue if it has no associated events

- For the POST /venues endpoint:
  - Accept a JSON object in the request body with venue details
  - Validate the input data
  - Create a new Venue entity
  - Save the venue to the database
  - Return a 201 Created status code with the created venue
  - Include the Location header with the URL to the new resource

- For the PUT /venues/{id} endpoint:
  - Accept a venue ID as a route parameter
  - Accept a JSON object in the request body with updated venue details
  - Validate the input data
  - Update the existing venue
  - Return a 200 OK status code with the updated venue
  - Return a 404 Not Found status code if the venue is not found

- For the DELETE /venues/{id} endpoint:
  - Accept a venue ID as a route parameter
  - Check if the venue has any associated events
  - If no events are associated, delete the venue
  - If events are associated, return an appropriate error
  - Return a 204 No Content status code on successful deletion
  - Return a 404 Not Found status code if the venue is not found

## Inputs and Outputs

**Inputs:**
- POST /venues - Venue data in request body
- PUT /venues/{id} - Venue ID in route, updated venue data in request body
- DELETE /venues/{id} - Venue ID in route

**Outputs:**
- POST /venues - 201 Created with venue data
- PUT /venues/{id} - 200 OK with updated venue data
- DELETE /venues/{id} - 204 No Content or appropriate error

## Reflection

Consider what should happen if someone tries to delete a venue that has scheduled events. Should you allow deletion and cascade to delete the events, or prevent deletion and return an error? Think about the appropriate validation rules for venue creation and updates.

Up Next: [Event Category Endpoints](./eventhorizon-category-endpoints.md)