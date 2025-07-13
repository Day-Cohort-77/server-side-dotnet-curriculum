# Category and Registration Models

## Feature Description

In this chapter, you'll define the models for event categories and registrations. These models will complete the core data structure of your application and establish the relationships between all entities.

## Requirements

- Create an EventCategory model with the following properties:
  - Id (int, primary key)
  - Name (string, required)
  - Description (string)

- Create a Registration model with the following properties:
  - Id (int, primary key)
  - EventId (int, foreign key, required)
  - UserProfileId (string, foreign key, required)
  - UserProfile (navigation property to the UserProfile model)
  - Event (navigation property to the Event model)
  - RegistrationDate (DateTime, required)
  - AttendeeCount (int, required)
  - Notes (string)

- Define the relationships between models:
  - An event belongs to one category
  - A category can have many events
  - An event can have many registrations
  - A registration belongs to one event
  - A registration belongs to one UserProfile
  - A UserProfile can have many registrations

- Add appropriate data annotations for validation:
  - Required fields
  - String length restrictions
  - Range validations for numeric fields (e.g., AttendeeCount should be positive)

## Reflection

Consider how registrations relate to event capacity. How will you ensure that an event doesn't exceed its maximum attendee limit? Think about what validation rules would be appropriate for the AttendeeCount property. Also, consider how to properly set up the relationship between Registration and UserProfile using the ASP.NET Core Identity framework.

Up Next: [Database Context](./eventhorizon-dbcontext.md)