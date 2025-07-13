# Database Context

## Feature Description

In this chapter, you'll create the database context that will manage the connection between your models and the database. The DbContext is a crucial component that allows Entity Framework to map your C# objects to database tables. You'll also configure ASP.NET Core Identity for user authentication and authorization.

## Requirements

- Create an EventHorizonDbContext class that inherits from IdentityDbContext<UserProfile>
- Add DbSet properties for all models:
  - DbSet<Event> Events
  - DbSet<Venue> Venues
  - DbSet<EventCategory> EventCategories
  - DbSet<Registration> Registrations

- Configure the OnModelCreating method to:
  - Call base.OnModelCreating(modelBuilder) to include Identity model configuration
  - Define any additional constraints or relationships
  - Configure Identity options as needed
  - Seed the database with initial data:
    - At least 3 venues with different capacities and locations
    - At least 5 event categories (e.g., Conference, Workshop, Concert, Networking, Seminar)
    - At least 10 events spread across different venues and categories
    - Events should have a mix of dates (past, present, future)
    - Default user roles (Admin and User)
    - A default admin user

- Create and apply an initial migration to set up the database schema

## Reflection

Think about what seed data would be most useful for testing various features. Consider creating events with different dates (past, current, future) to test date filtering. Also, consider how to handle the relationship between events and venues in your seed data to ensure referential integrity. For the Identity configuration, think about what password requirements would be appropriate for your application and how to create a default admin user with a secure password.

Up Next: [Organizing Endpoints](./eventhorizon-endpoints-organization.md)