# Jewelry Database

In this chapter, we'll focus on setting up and managing the database for our Jewelry Junction API. We'll explore how to create a more complex database schema, implement relationships between tables, and use Npgsql to interact with the database.

## Learning Objectives

By the end of this chapter, you should be able to:
- Design a more complex database schema with relationships
- Implement database initialization with SQL scripts
- Seed the database with initial data
- Query related data using SQL joins
- Implement data validation and constraints

## Reviewing the Database Schema

Let's review the database schema we defined in the previous chapter:

- **Products**: The jewelry items for sale
- **Metals**: The materials used to make the jewelry
- **Gemstones**: The precious stones used in the jewelry
- **ProductGemstones**: A join table for the many-to-many relationship between Products and Gemstones
- **Orders**: Customer purchases of products
- **OrderItems**: Items within an order
- **Customers**: People who place orders

This schema represents a typical e-commerce database with relationships between entities. Let's enhance it by adding some additional features.

## Enhancing the Database Schema

Let's enhance our database schema by adding a few more entities and relationships:

1. **Categories**: To categorize products (e.g., Rings, Necklaces, Bracelets)
2. **Discounts**: To apply discounts to products
3. **Reviews**: To allow customers to review products

For each of these entities, we'll define the table structure with appropriate columns and relationships to other tables. We'll implement these using SQL CREATE TABLE statements rather than Entity Framework Core migrations.

## Creating the Database Schema

To create our enhanced database schema, we'll write a SQL script that defines all the tables and their relationships. This script will:

1. Drop existing tables if they exist (for clean initialization)
2. Create tables for all entities in the correct order to respect foreign key constraints
3. Define primary keys, foreign keys, and other constraints

The script will include CREATE TABLE statements for:
- Categories
- Metals
- Gemstones
- Discounts
- Products
- ProductGemstones
- Customers
- Orders
- OrderItems
- Reviews

Each table will have appropriate columns with data types, constraints, and foreign key relationships.

## Implementing the Database Service

To interact with our database, we'll enhance our DatabaseService class with methods to:

1. Initialize the database using our SQL script
2. Execute SQL queries and commands
3. Map database results to our model objects
4. Handle transactions for operations that affect multiple tables

The DatabaseService will be the central point for all database operations in our application, providing a clean abstraction over the raw SQL queries.

## Seeding the Database

After creating the schema, we'll seed the database with initial data to work with. We'll create SQL INSERT statements for:

1. Categories (Rings, Necklaces, Bracelets, Earrings)
2. Metals (Gold, Silver, Platinum)
3. Gemstones (Diamond, Ruby, Emerald, Sapphire)
4. Discounts (Summer Sale, Clearance)
5. Sample products with relationships to metals, categories, and discounts
6. Sample product-gemstone relationships
7. Sample customers
8. Sample reviews

This seed data will provide a realistic starting point for testing our API endpoints.

## Implementing Data Access Methods

Now, we'll implement methods in our DatabaseService to perform common database operations:

1. **GetAllProducts**: Retrieve all products with their related data
2. **GetProductById**: Retrieve a single product by ID with its related data
3. **GetProductsByCategory**: Retrieve products filtered by category
4. **GetProductsByMetal**: Retrieve products filtered by metal
5. **GetProductsWithDiscount**: Retrieve products that have an active discount

These methods will use SQL queries with JOINs to retrieve related data in a single query, rather than making multiple database roundtrips.

## Implementing Data Validation

To ensure data integrity, we'll implement validation in our database service methods:

1. Check that required fields are provided
2. Validate numeric values (e.g., prices must be positive)
3. Verify that foreign key references exist before inserting or updating records
4. Implement business rules (e.g., order items must have valid products)

This validation will be performed before executing SQL commands to prevent invalid data from being stored in the database.

## Handling Transactions

For operations that affect multiple tables and need to be atomic (all succeed or all fail), we'll implement transaction handling:

1. Begin a transaction
2. Execute multiple SQL commands
3. Commit the transaction if all commands succeed
4. Roll back the transaction if any command fails

This ensures data consistency even when performing complex operations that update multiple related tables.

## Conclusion

In this chapter, you've learned how to set up a more complex database schema for the Jewelry Junction API. You've implemented the schema using SQL scripts, seeded the database with initial data, and created methods to interact with the database using Npgsql. You've also learned how to handle transactions and implement data validation to ensure data integrity.

In the next chapter, we'll implement endpoints for retrieving and managing products, including filtering, sorting, and pagination.

## Practice Exercise

Enhance your Jewelry Junction database by:
1. Adding a `Supplier` entity with columns for `id`, `name`, `contact_name`, `email`, and `phone`
2. Creating a relationship between `products` and `suppliers` (a product is supplied by one supplier)
3. Adding seed data for suppliers
4. Implementing validation for the supplier data
5. Creating methods in the DatabaseService to manage suppliers
6. Adding a method to calculate the total value of products supplied by a specific supplier