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

## Implementing a Product Details Endpoint

First, we'll implement an endpoint that retrieves detailed information about a product, including its metal, category, discount, gemstones, and reviews. This endpoint will:

1. Define a route handler for GET /products/{id}/details
2. Use a SQL query with multiple JOINs to retrieve all related data in a single query:
   - JOIN with metals table to get metal details
   - JOIN with categories table to get category details
   - LEFT JOIN with discounts table to get discount details (if any)
   - LEFT JOIN with product_gemstones and gemstones tables to get gemstone details
   - LEFT JOIN with reviews and customers tables to get review details

3. Calculate derived values:
   - Average rating based on reviews
   - Total gemstone value (sum of carats × price per carat)
   - Discounted price (if a discount is active)

4. Format the response to include all this information in a structured way

This comprehensive endpoint provides clients with all the information they need about a product in a single request.

## Implementing a Product Search Endpoint

Next, we'll implement a search endpoint that allows filtering products based on various criteria, including related data. This endpoint will:

1. Define a route handler for GET /products/search
2. Accept multiple query parameters for filtering:
   - Name/description (text search)
   - Product type
   - Category
   - Metal
   - Gemstone
   - Price range
   - Stock status
   - Discount status
   - Minimum rating

3. Build a dynamic SQL query with JOINs and WHERE clauses based on the provided parameters
4. Execute the query and format the response

This flexible search endpoint allows clients to find products that match specific criteria, making it easier to implement features like product filtering and search.

## Implementing a Sales Report Endpoint

To provide business insights, we'll implement a sales report endpoint that joins order data with product data. This endpoint will:

1. Define a route handler for GET /reports/sales
2. Accept date range parameters (start date, end date)
3. Accept a grouping parameter to specify how to aggregate the data
4. Use a SQL query with multiple JOINs to retrieve orders, order items, products, categories, and metals
5. Calculate summary metrics:
   - Total sales amount
   - Total items sold
   - Average order value
6. Group the data based on the specified parameter:
   - By category
   - By metal
   - By product
   - By date
7. Calculate metrics for each group
8. Format the response to include both summary metrics and grouped data

This reporting endpoint provides valuable business insights that can help with inventory planning, marketing decisions, and financial analysis.

## Implementing a Customer Order History Endpoint

To support customer account features, we'll implement an endpoint that retrieves a customer's order history. This endpoint will:

1. Define a route handler for GET /customers/{id}/orders
2. Verify the customer exists
3. Use a SQL query with JOINs to retrieve:
   - All orders for the customer
   - Order items for each order
   - Product details for each order item
4. Format the response to include:
   - Customer information
   - Order details (date, status, total amount)
   - Product information for each order item
   - Summary metrics (order count, total spent, average order value)

This endpoint provides a comprehensive view of a customer's purchase history, which can be used for account pages, order tracking, and customer support.

## Implementing a Product Recommendations Endpoint

To enhance the shopping experience, we'll implement a product recommendations endpoint based on a customer's order history. This endpoint will:

1. Define a route handler for GET /customers/{id}/recommendations
2. Analyze a customer's previous orders to determine preferences:
   - Identify categories they've purchased from
   - Identify metals they've shown interest in
3. Use a SQL query with JOINs to find products that:
   - Match these preferences
   - Haven't been purchased by the customer
   - Are in stock
4. Sort by rating to show the best products first
5. Limit to a reasonable number of recommendations (10)
6. Format the response to include product details and a reason for each recommendation

This recommendation engine provides personalized product suggestions that can increase cross-selling opportunities and enhance the customer experience.

## Optimizing Queries

When working with complex joins and queries, it's important to optimize them for performance. We'll explore several techniques:

1. **Selecting specific columns**: Instead of using SELECT *, specify only the columns you need
2. **Using indexes**: Ensure that columns used in JOIN and WHERE clauses are properly indexed
3. **Filtering early**: Apply WHERE clauses before JOINs to reduce the number of rows processed
4. **Using appropriate join types**: Choose the right type of join for each relationship
5. **Implementing pagination**: Use LIMIT and OFFSET to retrieve only the data needed for the current page
6. **Avoiding subqueries when possible**: Use JOINs instead of subqueries when appropriate

We'll update our product search endpoint to demonstrate these optimization techniques:
- Adding pagination parameters (page number, page size)
- Using LIMIT and OFFSET in the SQL query
- Including pagination metadata in the response

These optimizations ensure that our API remains performant even with large datasets and complex queries.

## Conclusion

In this chapter, you've learned how to join related data in the Jewelry Junction API. You've implemented endpoints that retrieve data from multiple related tables, perform complex queries, and format the results in a meaningful way. You've also learned how to optimize queries for performance.

These techniques are essential for building robust and efficient APIs that can handle complex data relationships and provide meaningful insights to clients.

## Practice Exercise

Enhance your Jewelry Junction API by:
1. Implementing an endpoint that retrieves the top-selling products for a given time period
2. Creating an endpoint that provides inventory status reports (e.g., products with low stock, out-of-stock products)
3. Implementing an endpoint that calculates the total value of the inventory
4. Creating an endpoint that provides customer insights (e.g., top customers, customer purchase patterns)
5. Implementing an endpoint that generates a report of products that haven't sold in a given time period