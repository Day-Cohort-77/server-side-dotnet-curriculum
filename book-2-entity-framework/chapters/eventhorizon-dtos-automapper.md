# DTOs and AutoMapper

## Feature Description

In this chapter, you'll create Data Transfer Objects (DTOs) for your API responses and set up AutoMapper to map between your domain models and DTOs.

## Requirements

- Create DTO classes in the DTO directory:
  - VenueDto
  - EventDto
  - EventCategoryDto
  - RegistrationDto
  - UserDto (for later use)

- Each DTO should:
  - Include only the properties needed for API responses
  - Exclude sensitive or unnecessary information
  - Include appropriate nested DTOs for related entities
  - Add any calculated properties that might be useful

- Set up AutoMapper:
  - Create a MappingProfile class in the Mappings directory
  - Configure mappings between domain models and DTOs
  - Register AutoMapper in Program.cs

- Consider creating specialized DTOs for specific operations:
  - CreateEventDto
  - UpdateEventDto
  - EventDetailDto (with more information than the basic EventDto)

## Reflection

Think about what information should be exposed in your API responses. Not all properties from your domain models need to be included in DTOs. Consider adding calculated properties to your DTOs that might be useful for clients, such as a formatted date string or a full address property that combines address components.

Up Next: [Venue Endpoints - Read Operations](./eventhorizon-venue-read.md)