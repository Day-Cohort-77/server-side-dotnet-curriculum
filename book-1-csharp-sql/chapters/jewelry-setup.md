# Project Setup

In this chapter, we'll set up the Jewelry Junction project, a minimal API for managing a jewelry store's inventory and orders. This project will build on the concepts you've learned in the previous tracks, applying them to a new domain.

## Learning Objectives

By the end of this chapter, you should be able to:
- Set up a new minimal API project for Jewelry Junction
- Understand the project requirements and domain
- Define the initial data models
- Create a basic project structure
- Implement a simple endpoint to verify the setup

## Project Overview

Jewelry Junction is an online jewelry store that needs an API to manage its inventory and customer orders. The API will allow the store to:

- Manage products (rings, necklaces, bracelets, etc.)
- Track inventory
- Process customer orders
- Manage metals (gold, silver, platinum) and gemstones (diamonds, rubies, emeralds, etc.)
- Generate reports on sales and inventory

Throughout this track, you'll build this API step by step, learning how to implement various features and best practices.

## Setting Up the Project

To begin, you'll create a new minimal API project using the ASP.NET Core framework. This involves:

1. Creating a new project using the webapi template with the minimal API option
2. Setting up the project directory structure with folders for Models and Services
3. Adding the Npgsql package to connect to PostgreSQL
4. Opening the project in your preferred IDE

The minimal API approach provides a streamlined way to build APIs with less boilerplate code than traditional controllers.

## Understanding the Domain

Before diving into implementation, it's important to understand the domain of our application. Jewelry Junction deals with several key entities:

- **Products**: The jewelry items for sale (rings, necklaces, bracelets, etc.)
- **Metals**: The materials used to make the jewelry (gold, silver, platinum, etc.)
- **Gemstones**: The precious stones used in the jewelry (diamonds, rubies, emeralds, etc.)
- **Categories**: The types of jewelry (rings, necklaces, bracelets, etc.)
- **Orders**: Customer purchases of products
- **Customers**: People who place orders

Understanding these entities and their relationships is crucial for designing an effective database schema.

## Defining the Data Models

Based on our domain understanding, we need to create several model classes to represent our entities:

1. **Product**: Represents jewelry items with properties like Id, Name, Description, Price, StockQuantity, and Type. It will have foreign key references to Metal, Category, and optionally Discount.

2. **Metal**: Represents materials like gold, silver, and platinum with properties like Id, Name, and PricePerGram.

3. **Gemstone**: Represents precious stones like diamonds, rubies, and emeralds with properties like Id, Name, and PricePerCarat.

4. **ProductGemstone**: A join entity for the many-to-many relationship between Products and Gemstones, with properties like Id, ProductId, GemstoneId, and Carats.

5. **Category**: Represents product categories with properties like Id, Name, and Description.

6. **Order**: Represents customer orders with properties like Id, OrderDate, TotalAmount, and Status. It will have a foreign key reference to Customer.

7. **OrderItem**: Represents items within an order with properties like Id, OrderId, ProductId, Quantity, and UnitPrice.

8. **Customer**: Represents people who place orders with properties like Id, FirstName, LastName, Email, Phone, and Address.

These models will be simple POCO (Plain Old CLR Object) classes without navigation properties, as we'll be using direct SQL queries instead of an ORM.

## Setting Up Database Connection

To connect to our PostgreSQL database, we'll:

1. Add a connection string to the appsettings.json file
2. Create a DatabaseService class that will handle all database operations
3. Implement methods to create a database connection and execute SQL commands

The DatabaseService will be the central point for all database interactions in our application.

## Creating the Database Schema

Next, we'll create the database schema using SQL scripts. This involves:

1. Creating a SQL script to define all the tables and relationships
2. Implementing a method in the DatabaseService to initialize the database
3. Ensuring the database is created when the application starts

The SQL script will define tables for all our entities and establish the necessary relationships between them.

## Seeding the Database

To have some initial data to work with, we'll seed the database with:

1. Categories (Rings, Necklaces, Bracelets, Earrings)
2. Metals (Gold, Silver, Platinum)
3. Gemstones (Diamond, Ruby, Emerald, Sapphire)
4. Sample products
5. Sample customers

This seed data will be added using SQL INSERT statements executed by the DatabaseService.

## Implementing Basic Endpoints

Finally, we'll implement some basic endpoints to verify our setup:

1. A welcome endpoint at the root URL
2. An endpoint to get all products
3. An endpoint to get a product by ID

These endpoints will allow us to test that our API is working correctly and can retrieve data from the database.

## Running and Testing the API

After setting up the project, we'll:

1. Run the API using the dotnet run command
2. Access the Swagger UI to explore and test our endpoints
3. Test the GET /products endpoint to retrieve all products
4. Test the GET /products/{id} endpoint to retrieve a specific product

Swagger provides a user-friendly interface for testing our API without needing additional tools.

## Conclusion

In this chapter, you've learned how to set up the Jewelry Junction project, define the data models, create a database schema, and implement basic endpoints. You've also learned how to seed the database with initial data and test the API.

In the next chapter, we'll expand the API by implementing endpoints for managing the jewelry database, including creating, updating, and deleting products.

## Practice Exercise

Enhance your Jewelry Junction API by:
1. Adding endpoints for managing metals (GET, POST, PUT, DELETE)
2. Adding endpoints for managing gemstones (GET, POST, PUT, DELETE)
3. Implementing a search endpoint that allows filtering products by type, metal, or price range
4. Adding validation to ensure that product prices are positive and stock quantities are non-negative
5. Creating a simple endpoint that returns statistics about the inventory (total number of products, average price, etc.)