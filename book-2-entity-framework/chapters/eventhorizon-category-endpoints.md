# Event Category Endpoints

## Feature Description

In this chapter, you'll implement endpoints to manage event categories. These endpoints will provide complete CRUD functionality for event categories.

## Requirements

- Implement the following endpoints in CategoryEndpoints.cs:
  - GET /categories - Returns all categories
  - GET /categories/{id} - Returns a specific category
  - POST /categories - Creates a new category
  - PUT /categories/{id} - Updates a category
  - DELETE /categories/{id} - Deletes a category if it has no associated events

- For the GET /categories endpoint:
  - Return a list of all categories
  - Map the EventCategory entities to EventCategoryDto objects using AutoMapper
  - Return a 200 OK status code with the list of categories

- For the GET /categories/{id} endpoint:
  - Accept a category ID as a route parameter
  - Return the category with the specified ID
  - Map the EventCategory entity to an EventCategoryDto using AutoMapper
  - Return a 200 OK status code if the category is found
  - Return a 404 Not Found status code if the category is not found

- For the POST /categories endpoint:
  - Accept a JSON object in the request body with category details
  - Validate the input data
  - Create a new EventCategory entity
  - Save the category to the database
  - Return a 201 Created status code with the created category
  - Include the Location header with the URL to the new resource

- For the PUT /categories/{id} endpoint:
  - Accept a category ID as a route parameter
  - Accept a JSON object in the request body with updated category details
  - Validate the input data
  - Update the existing category
  - Return a 200 OK status code with the updated category
  - Return a 404 Not Found status code if the category is not found

- For the DELETE /categories/{id} endpoint:
  - Accept a category ID as a route parameter
  - Check if the category has any associated events
  - If no events are associated, delete the category
  - If events are associated, return an appropriate error
  - Return a 204 No Content status code on successful deletion
  - Return a 404 Not Found status code if the category is not found

## Inputs and Outputs

**Inputs:**
- GET /categories - No required inputs
- GET /categories/{id} - Category ID as a route parameter
- POST /categories - Category data in request body
- PUT /categories/{id} - Category ID in route, updated category data in request body
- DELETE /categories/{id} - Category ID in route

**Outputs:**
- GET /categories - A JSON array of category objects
- GET /categories/{id} - A JSON object representing a single category
- POST /categories - 201 Created with category data
- PUT /categories/{id} - 200 OK with updated category data
- DELETE /categories/{id} - 204 No Content or appropriate error

## Reflection

Think about what validation would be appropriate for category names. Should you allow duplicate category names? Consider how to handle the case where someone tries to delete a category that has events assigned to it.

Up Next: [Event Endpoints - Basic Operations](./eventhorizon-event-basic.md)