# User, Event, and Venue Models

## Feature Description

In this chapter, you'll define the core data models that will represent users, events, and venues in the system. These models will form the foundation of your database structure and will be used throughout the application. You'll also set up the UserProfile model using ASP.NET Core Identity.

## Requirements

- Create a UserProfile model that extends IdentityUser:
  - FirstName (string, required)
  - LastName (string, required)
  - JoinDate (DateTime, required)
  - IsActive (bool, default true)
  - Add navigation property for Registrations

- Create an Event model with the following properties:
  - Id (int, primary key)
  - Name (string, required)
  - Description (string)
  - DateTime (DateTime, required)
  - MaxAttendees (int, required)
  - VenueId (int, foreign key)
  - EventCategoryId (int, foreign key)
  - IsActive (bool, default true)

- Create a Venue model with the following properties:
  - Id (int, primary key)
  - Name (string, required)
  - Address (string, required)
  - City (string, required)
  - State (string, required)
  - ZipCode (string, required)
  - MaxCapacity (int, required)
  - Description (string)
  - IsActive (bool, default true)

- Define the relationship between events and venues:
  - An event belongs to one venue
  - A venue can host many events

- Add appropriate data annotations for validation:
  - Required fields
  - String length restrictions
  - Range validations for numeric fields

## Reflection

Think about what constraints would be reasonable for each property. For example, what should be the minimum and maximum values for MaxAttendees? What string length limits make sense for Name and Description fields? For the UserProfile model, consider what additional properties beyond the base IdentityUser properties would be useful for your application.

Up Next: [Category and Registration Models](./eventhorizon-category-registration-models.md)