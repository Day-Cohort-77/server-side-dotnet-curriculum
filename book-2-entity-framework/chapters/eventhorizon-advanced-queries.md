# Advanced Queries

## Feature Description

In this chapter, you'll implement advanced query endpoints for the API. These endpoints will provide specialized views of the data that go beyond basic CRUD operations.

## Requirements

- Implement the following endpoints:
  - GET /events/upcoming - Returns upcoming events
  - GET /events/popular - Returns events with the most registrations
  - GET /venues/popular - Returns venues that host the most events
  - GET /events/daterange?start={startDate}&end={endDate} - Returns events within a date range

- For the GET /events/upcoming endpoint:
  - Return events that are scheduled to occur in the future
  - Order events by date (soonest first)
  - Limit results to a reasonable number (e.g., 10)
  - Map the Event entities to EventDto objects using AutoMapper
  - Return a 200 OK status code with the list of events
  - Consider adding a "days" query parameter to specify how many days in the future to look

- For the GET /events/popular endpoint:
  - Return events ordered by the number of registrations (most registrations first)
  - Limit results to a reasonable number (e.g., 10)
  - Include the registration count in the response
  - Map the results to a specialized DTO that includes registration count
  - Return a 200 OK status code with the list of events

- For the GET /venues/popular endpoint:
  - Return venues ordered by the number of events hosted (most events first)
  - Limit results to a reasonable number (e.g., 10)
  - Include the event count in the response
  - Map the results to a specialized DTO that includes event count
  - Return a 200 OK status code with the list of venues

- For the GET /events/daterange endpoint:
  - Accept start and end dates as query parameters
  - Return events that occur within the specified date range
  - Order events by date
  - Map the Event entities to EventDto objects using AutoMapper
  - Return a 200 OK status code with the list of events
  - Return appropriate error messages for invalid date formats

## Inputs and Outputs

**Inputs:**
- GET /events/upcoming - Optional "days" query parameter
- GET /events/popular - No required inputs
- GET /venues/popular - No required inputs
- GET /events/daterange - "start" and "end" date query parameters

**Outputs:**
- GET /events/upcoming - 200 OK with events array
- GET /events/popular - 200 OK with events array including registration counts
- GET /venues/popular - 200 OK with venues array including event counts
- GET /events/daterange - 200 OK with events array

## Reflection

Think about what sorting and filtering options would be most useful. Consider using LINQ to create efficient queries that filter and sort the data at the database level rather than in memory. For the popular events and venues endpoints, you'll need to use grouping and counting operations in your LINQ queries.

Up Next: [Final Requirements](./eventhorizon-final.md)