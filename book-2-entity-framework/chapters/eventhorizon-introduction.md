# Introduction to EventHorizon

☀️ 🚀 ☀️ 🚀 ☀️ 🚀 ☀️ 🚀 ☀️ 🚀

It is time for you to implement your own API with no code guidance at all. The chapters for this project will provide you with requirements and some things to reflect on, but not code.

☀️ 🚀 ☀️ 🚀 ☀️ 🚀 ☀️ 🚀 ☀️ 🚀

## Project Overview


EventHorizon is an event planning API that will help event organizers manage events, venues, attendees, and registrations. This project will allow you to apply the Entity Framework concepts you've learned in a new context, along with user authentication and authorization using ASP.NET Core Identity.

## Feature Description

In this chapter, you'll set up the EventHorizon project with the necessary structure and dependencies. This will establish the foundation for all future features.

## Requirements

- Create a new ASP.NET Core Minimal API project named "EventHorizon"
- Set up the following directory structure:
  - DTO: For data transfer objects
  - Models: For database entity models
  - Endpoints: For API endpoint definitions
  - Data: For DbContext and related services
  - Mappings: For AutoMapper configurations
- Install the required NuGet packages:
  - Npgsql.EntityFrameworkCore.PostgreSQL
  - Microsoft.EntityFrameworkCore.Design
  - Microsoft.AspNetCore.Identity.EntityFrameworkCore
  - AutoMapper.Extensions.Microsoft.DependencyInjection
- Configure a database connection using user secrets
- Set up debugging configuration

## Reflection

Consider how the project structure will support separation of concerns as the application grows. Think about what naming conventions will make your code more maintainable. The inclusion of ASP.NET Core Identity will allow you to implement user authentication and authorization, which is essential for a secure event planning API.

Up Next: [User, Event, and Venue Models](./eventhorizon-models.md)