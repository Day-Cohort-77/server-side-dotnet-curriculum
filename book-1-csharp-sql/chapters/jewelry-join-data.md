# Joining Related Data

In this chapter, we'll explore how to join related data in our Jewelry Junction API. We'll implement endpoints that retrieve data from multiple related tables, perform complex queries, and format the results in a meaningful way.

## Learning Objectives

By the end of this chapter, you should be able to:
- Understand the concept of joining related data in SQL
- Implement endpoints that retrieve data from multiple related tables
- Use SQL joins to perform complex queries
- Format joined data in a meaningful way
- Optimize queries for performance

## Understanding SQL Joins

SQL provides several types of joins to combine data from multiple tables:
- **INNER JOIN**: Returns records that have matching values in both tables
- **LEFT JOIN**: Returns all records from the left table and matching records from the right table
- **RIGHT JOIN**: Returns all records from the right table and matching records from the left table
- **FULL JOIN**: Returns all records when there is a match in either the left or right table

In our Jewelry Junction API, we'll primarily use INNER JOINs and LEFT JOINs to retrieve related data.

## Updating the Product Details Endpoint

Your current single product endpoint retrieves the basic information about the product, but now you will utilize the powerful `JOIN` operation in SQL to provide more details to the response body.

1. Open the route handler for GET `/products/{id}`
2. Update the existing SQL query with JOINs to retrieve all related data in a single query:
   - JOIN with metals table to get metal details
   - JOIN with gemstones table to get gemstone details
   - JOIN with styles table to get style details
   ```sql
   --Don't ever use the `*` character in your SELECT clause
   SELECT
       p.id AS product_id,
       p.name AS product_name,
       p.description AS product_description,
       p.price AS product_price,
       m.id AS metal_id,
       m.name AS metal_name,
       m.price_per_gram AS metal_price_per_gram,
       g.id AS gemstone_id,
       g.name AS gemstone_name,
       g.price_per_carat AS gemstone_price_per_carat,
       s.id AS style_id,
       s.name AS style_name
   FROM products p
   INNER JOIN metals m ON p.metal_id = m.id
   INNER JOIN gemstones g ON p.gemstone_id = g.id
   INNER JOIN styles s ON p.style_id = s.id
   WHERE p.id = @productId;
   ```

3. Calculate derived values:
   - Total metal value (based on metal price per gram)
   - Total gemstone value (based on gemstone price per carat)
   - Total product value (base price + metal value + gemstone value)

4. Format the response to include all this information in a structured way

This comprehensive endpoint provides clients with all the information they need about a product in a single request.

## Implementing a Product Search Endpoint

Next, we'll implement a search endpoint that allows filtering products based on various criteria, including related data. This endpoint will:

1. Define a route handler for GET `/products/search`
2. Accept multiple query parameters for filtering:
   - Name/description (text search)
   - Metal
   - Gemstone
   - Style
   - Price range

3. Build a dynamic SQL query with JOINs and WHERE clauses based on the provided parameters
4. Execute the query and format the response

This flexible search endpoint allows clients to find products that match specific criteria, making it easier to implement features like product filtering and search.

## Implementing a Products by Style Endpoint

To help customers browse products by style, we'll implement an endpoint that retrieves all products with a specific style:

1. Define a route handler for GET `/styles/{id}/products`
2. Accept optional filter parameters (metal, price range)
3. Use a SQL query with JOINs to retrieve products with the specified style
4. Include related metal and gemstone information
5. Format the response to include:
   - Style information
   - List of products with details
   - Summary metrics (product count, price range)

This endpoint makes it easy for clients to display products grouped by style, which is a common way for customers to browse jewelry.

## Implementing a Products by Metal and Gemstone Endpoint

To help customers find products with specific material combinations, we'll implement an endpoint that retrieves products with a specific metal and gemstone:

1. Define a route handler for GET `/products/materials`
2. Accept required parameters for metal ID and gemstone ID
3. Accept optional parameters for style and price range
4. Use a SQL query with JOINs to retrieve matching products
5. Format the response to include:
   - Metal and gemstone information
   - List of products with details
   - Summary metrics (product count, price range)

This endpoint makes it easy for clients to display products filtered by specific material combinations, which is useful for customers with specific preferences.

## Implementing a Product Comparison Endpoint

To help customers compare different products, we'll implement an endpoint that retrieves detailed information for multiple products:

1. Define a route handler for GET `/products/compare`
2. Accept a list of product IDs as a query parameter
3. Use a SQL query with JOINs to retrieve detailed information for each product
4. Calculate comparative metrics:
   - Price difference
   - Value difference (total value of each product)
   - Material quality comparison
5. Format the response to include:
   - Detailed information for each product
   - Comparative metrics
   - Recommendation based on value

This endpoint makes it easy for clients to display product comparisons, which can help customers make informed purchasing decisions.

## Conclusion

In this chapter, you've learned how to join related data in the Jewelry Junction API. You've implemented endpoints that retrieve data from multiple related tables, perform complex queries, and format the results in a meaningful way. You've also learned how to optimize queries for performance.

These techniques are essential for building robust and efficient APIs that can handle complex data relationships and provide meaningful insights to clients.

## Practice Exercise

Enhance your Jewelry Junction API by:
1. Implementing an endpoint that retrieves products sorted by price within a specific style
2. Creating an endpoint that provides products with similar characteristics to a given product
3. Implementing an endpoint that calculates the total value of all products
4. Creating an endpoint that retrieves products with a specific metal across all styles
5. Implementing an endpoint that retrieves the most expensive and least expensive products for each style