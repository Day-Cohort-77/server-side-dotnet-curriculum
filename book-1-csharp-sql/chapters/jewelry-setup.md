# Project Setup

In this chapter, you will set up the Jewelry Junction project, a minimal API for managing a jewelry store's inventory and orders. This project will build on the concepts you've learned in the previous tracks, applying them to a new domain.

## Learning Objectives

By the end of this chapter, you should be able to:
- Set up a new minimal API project for Jewelry Junction
- Understand the project requirements and domain
- Define the initial data models
- Create a basic project structure
- Implement a simple endpoint to verify the setup

## Project Overview

Jewelry Junction is an online jewelry store that needs an API to manage its products and customer orders. The API will allow the store to:

- Manage products (rings, necklaces, bracelets, etc.)
- Process customer orders
- Manage metals (gold, silver, platinum)
- Manage gemstones (diamonds, rubies, emeralds, etc.)
- Manage styles (classic, modern, art deco, minimalist)

Throughout this track, you'll build this API step by step, learning how to implement various features and best practices.

## Setting Up the Project

To begin, you'll create a new minimal API project using the ASP.NET Core framework. You can refer to the content from previous projects for these steps.

1. Creating a new project using the `dotnet new webapi` template with the minimal API option
2. Setting up the project directory structure with folders for Endpoints, Models and Services
3. Adding the Npgsql package to connect to PostgreSQL
4. Opening the project in VS Code

The minimal API approach provides a streamlined way to build APIs with less boilerplate code than traditional controllers.

## Understanding the Domain

Before diving into implementation, it's important to understand the domain of our application. Jewelry Junction deals with several key entities:

- **Products**: The jewelry items for sale (rings, necklaces, bracelets, etc.)
- **Metals**: The materials used to make the jewelry (gold, silver, platinum, etc.)
- **Gemstones**: The precious stones used in the jewelry (diamonds, rubies, emeralds, etc.)
- **Styles**: The design styles of jewelry (classic, modern, art deco, minimalist)
- **Orders**: Customer purchases of products

Understanding these entities and their relationships is crucial for designing an effective database schema.

## Creating the Database

In the next chapter, you will follow the steps for creating the database, the tables, and the `DatabaseService`. Once created, you will verify that the tables exist by creating a new connection with the **SQLTools** VSCode extension.

## Defining the Data Models

Based on your domain understanding, you will need to create several model classes to represent your entities:

1. **Product**: Represents jewelry items with properties like `Id`, `Name`, `Description`, and `Price`. It will have foreign key references to Metal, Gemstone, and Style.
2. **Metal**: Represents materials like gold, silver, and platinum with properties like `Id`, `Name`, and `PricePerGram`.
3. **Gemstone**: Represents precious stones like diamonds, rubies, and emeralds with properties like `Id`, `Name`, and `PricePerCarat`.
4. **Style**: Represents design styles like classic, modern, art deco, and minimalist with properties like `Id` and `Name`.
5. **Order**: Represents customer orders with properties like `Id`, `OrderDate`, `CustomerName`, and `Status`.
6. **OrderItem**: Represents items within an order with properties like `Id`, `OrderId`, `ProductId`, `Quantity`, and `UnitPrice`.

Refer to your model classes in the previous project as guidance for creating these. Use dbdiagram.io to make an ERD for this project. Your teammates and coaching team are great resources to review this with you.

## Implementing Basic Endpoints

Finally, you will implement some basic endpoints to verify your setup:

1. A welcome endpoint at the root URL
2. An endpoint to get all products
3. An endpoint to get a product by ID

These endpoints will allow us to test that our API is working correctly and can retrieve data from the database.

## Running and Testing the API

After setting up the project, you will:

1. Create the `launch.json` and `tasks.json` file for configuring your debugger
2. Open Yaak for testing your functionality
3. Test the GET `/products` endpoint to retrieve all products
4. Test the GET `/products/{id}` endpoint to retrieve a specific product

## Next Steps

In the next chapter you will create all assets needed for creating and seeding your database.

[Database setup >](./jewelry-database.md)