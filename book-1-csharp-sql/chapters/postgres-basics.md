# PostgreSQL Basics

In this chapter, we'll explore the basics of PostgreSQL, a powerful open-source relational database management system. You'll learn how to install PostgreSQL, create databases and tables, and perform basic SQL operations.

## Learning Objectives

By the end of this chapter, you should be able to:
- Understand what PostgreSQL is and why it's used
- Install and configure PostgreSQL on your machine
- Use pgAdmin to interact with PostgreSQL
- Create databases and tables
- Understand basic data types in PostgreSQL
- Perform basic SQL operations (SELECT, INSERT, UPDATE, DELETE)

## What is PostgreSQL?

PostgreSQL (often referred to as "Postgres") is a powerful, open-source object-relational database system. It has more than 30 years of active development and a proven architecture that has earned it a strong reputation for reliability, feature robustness, and performance.

Key features of PostgreSQL include:
- ACID compliance (Atomicity, Consistency, Isolation, Durability)
- Support for complex data types
- Robust transaction support
- Extensibility
- Multi-version concurrency control
- Strong standards compliance

## Installing PostgreSQL

If you haven't already installed PostgreSQL as part of the Book 1 installations, follow these steps:

### Windows
1. Visit the [PostgreSQL download page](https://www.postgresql.org/download/windows/)
2. Download the installer from EnterpriseDB
3. Run the installer and follow the prompts
4. Remember the password you set for the postgres user
5. Keep the default port (5432)
6. Complete the installation

### macOS
1. The easiest way to install PostgreSQL on macOS is using Homebrew
2. If you don't have Homebrew installed, install it first:
   ```
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```
3. Install PostgreSQL:
   ```
   brew install postgresql
   ```
4. Start the PostgreSQL service:
   ```
   brew services start postgresql
   ```

### Linux
1. For Ubuntu/Debian:
   ```
   sudo apt update
   sudo apt install postgresql postgresql-contrib
   ```
2. For Fedora:
   ```
   sudo dnf install postgresql-server postgresql-contrib
   sudo systemctl enable postgresql
   sudo postgresql-setup --initdb --unit postgresql
   sudo systemctl start postgresql
   ```

## Installing pgAdmin

pgAdmin is a popular administration and management tool for PostgreSQL. It provides a graphical interface to help you manage your databases.

1. Visit the [pgAdmin download page](https://www.pgadmin.org/download/)
2. Select your operating system
3. Download and install the latest version
4. Launch pgAdmin after installation
5. You'll be prompted to set a master password for pgAdmin
6. Connect to your PostgreSQL server using the credentials you set during PostgreSQL installation

## Connecting to PostgreSQL

Once you have PostgreSQL and pgAdmin installed, you can connect to your PostgreSQL server:

1. Open pgAdmin
2. In the left panel, right-click on "Servers" and select "Create" > "Server..."
3. In the "General" tab, give your server a name (e.g., "LocalPostgres")
4. In the "Connection" tab, enter the following information:
   - Host name/address: localhost
   - Port: 5432
   - Maintenance database: postgres
   - Username: postgres
   - Password: (the password you set during installation)
5. Click "Save" to connect to your PostgreSQL server

## Creating a Database

Now that you're connected to your PostgreSQL server, let's create a database:

1. In pgAdmin, right-click on "Databases" under your server and select "Create" > "Database..."
2. Enter a name for your database (e.g., "ExtraVert")
3. Click "Save" to create the database

You can also create a database using SQL:

```sql
CREATE DATABASE extravert;
```

To execute this SQL command in pgAdmin:
1. Right-click on your server and select "Query Tool"
2. Enter the SQL command in the query editor
3. Click the "Execute" button (or press F5)

## Understanding PostgreSQL Data Types

PostgreSQL supports a wide range of data types. Here are some common ones:

| Data Type | Description | Example |
|-----------|-------------|---------|
| INTEGER | Whole number | 42 |
| SERIAL | Auto-incrementing integer | 1, 2, 3, ... |
| NUMERIC(p,s) | Exact numeric with precision p and scale s | 123.45 |
| VARCHAR(n) | Variable-length character string with limit n | 'Hello' |
| TEXT | Variable-length character string with no limit | 'A long text...' |
| BOOLEAN | Logical Boolean (true/false) | true |
| DATE | Calendar date (year, month, day) | '2025-05-01' |
| TIME | Time of day | '14:30:00' |
| TIMESTAMP | Date and time | '2025-05-01 14:30:00' |
| UUID | Universally unique identifier | 'a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11' |
| JSON | JSON data | '{"name": "John", "age": 30}' |
| JSONB | Binary JSON data (more efficient) | '{"name": "John", "age": 30}' |

## Creating Tables

Tables are the basic structure for storing data in a relational database. Let's create a table for our plants:

```sql
CREATE TABLE plants (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    species VARCHAR(100) NOT NULL,
    light_needs VARCHAR(50),
    water_needs VARCHAR(50),
    price NUMERIC(10, 2) NOT NULL,
    acquisition_date DATE NOT NULL DEFAULT CURRENT_DATE
);
```

This SQL command creates a table called "plants" with the following columns:
- `id`: A unique identifier for each plant (auto-incrementing)
- `name`: The name of the plant (required)
- `species`: The species of the plant (required)
- `light_needs`: The light requirements of the plant
- `water_needs`: The water requirements of the plant
- `price`: The price of the plant (required)
- `acquisition_date`: The date the plant was acquired (defaults to the current date)

To create this table in pgAdmin:
1. Connect to your "ExtraVert" database
2. Open the Query Tool
3. Enter the SQL command
4. Click "Execute"

## Basic SQL Operations

Now that we have a table, let's learn how to perform basic SQL operations: SELECT, INSERT, UPDATE, and DELETE.

### INSERT

The INSERT statement adds new rows to a table:

```sql
INSERT INTO plants (name, species, light_needs, water_needs, price)
VALUES ('Snake Plant', 'Sansevieria trifasciata', 'Low to bright indirect', 'Low', 15.99);

INSERT INTO plants (name, species, light_needs, water_needs, price)
VALUES ('Monstera', 'Monstera deliciosa', 'Bright indirect', 'Moderate', 29.99);

INSERT INTO plants (name, species, light_needs, water_needs, price)
VALUES ('Peace Lily', 'Spathiphyllum', 'Low to bright indirect', 'Moderate', 19.99);
```

Note that we didn't specify the `id` or `acquisition_date` columns. The `id` is automatically generated because it's a SERIAL column, and the `acquisition_date` defaults to the current date.

### SELECT

The SELECT statement retrieves data from a table:

```sql
-- Select all columns from all rows
SELECT * FROM plants;

-- Select specific columns
SELECT name, species, price FROM plants;

-- Select with a condition
SELECT * FROM plants WHERE price < 20.00;

-- Select with multiple conditions
SELECT * FROM plants WHERE light_needs LIKE '%indirect%' AND water_needs = 'Low';

-- Select with ordering
SELECT * FROM plants ORDER BY price DESC;

-- Select with limit
SELECT * FROM plants LIMIT 2;
```

### UPDATE

The UPDATE statement modifies existing rows in a table:

```sql
-- Update a single row
UPDATE plants
SET price = 17.99
WHERE name = 'Snake Plant';

-- Update multiple rows
UPDATE plants
SET water_needs = 'Moderate to high'
WHERE water_needs = 'Moderate';

-- Update multiple columns
UPDATE plants
SET price = 24.99,
    light_needs = 'Full sun to partial shade'
WHERE name = 'Monstera';
```

### DELETE

The DELETE statement removes rows from a table:

```sql
-- Delete a single row
DELETE FROM plants
WHERE name = 'Peace Lily';

-- Delete multiple rows
DELETE FROM plants
WHERE price < 20.00;

-- Delete all rows (be careful!)
DELETE FROM plants;
```

## Creating Tables with Relationships

In a relational database, tables can be related to each other. Let's create a table for plant categories and relate it to our plants table:

```sql
-- Create the categories table
CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    description TEXT
);

-- Add some categories
INSERT INTO categories (name, description)
VALUES ('Flowering', 'Plants that produce flowers');

INSERT INTO categories (name, description)
VALUES ('Foliage', 'Plants grown primarily for their leaves');

INSERT INTO categories (name, description)
VALUES ('Succulent', 'Plants that store water in their leaves or stems');

-- Add a category_id column to the plants table
ALTER TABLE plants
ADD COLUMN category_id INTEGER REFERENCES categories(id);

-- Update the plants with their categories
UPDATE plants
SET category_id = 3  -- Succulent
WHERE name = 'Snake Plant';

UPDATE plants
SET category_id = 2  -- Foliage
WHERE name = 'Monstera';

UPDATE plants
SET category_id = 1  -- Flowering
WHERE name = 'Peace Lily';
```

Now we can select plants with their categories:

```sql
SELECT p.name, p.species, c.name AS category
FROM plants p
JOIN categories c ON p.category_id = c.id;
```

## Conclusion

In this chapter, you've learned the basics of PostgreSQL and SQL. You've installed PostgreSQL and pgAdmin, created a database and tables, and performed basic SQL operations. These skills will be essential as we continue to build our ExtraVert Garden application and connect it to a database.

## Next Steps

In the next chapter, we'll explore how to connect our C# application to a PostgreSQL database using ADO.NET and the Npgsql provider.

## Practice Exercise

Enhance your understanding of PostgreSQL by:
1. Creating a new table called `plant_care` with columns for `id`, `plant_id`, `care_date`, `care_type` (e.g., 'Watering', 'Fertilizing'), and `notes`
2. Adding foreign key constraints to relate the `plant_care` table to the `plants` table
3. Inserting some sample care records for your plants
4. Writing SELECT queries to retrieve plants with their care records
5. Writing a query to find plants that haven't been watered in the last 7 days