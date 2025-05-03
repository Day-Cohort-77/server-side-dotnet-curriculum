# Get All Products

In this chapter, we'll implement and enhance the endpoint for retrieving all products in our Jewelry Junction API. We'll explore how to optimize queries, implement filtering, sorting, and pagination, and format the response data.

## Learning Objectives

By the end of this chapter, you should be able to:
- Implement a GET endpoint to retrieve all products
- Add filtering capabilities to the endpoint
- Implement sorting options
- Add pagination to handle large result sets
- Format and shape the response data
- Optimize database queries for performance

## Implementing the Basic Endpoint

We'll start by implementing a basic endpoint for retrieving all products. This endpoint will:

1. Define a route handler for GET /products
2. Use the DatabaseService to execute a SQL query that retrieves all products
3. Map the database results to Product objects
4. Return the list of products as a JSON response

The SQL query will be simple at first, just selecting all columns from the products table.

## Adding Filtering Capabilities

To make our API more flexible, we'll enhance the endpoint to support filtering based on various criteria:

1. Metal (filter by metal_id)
2. Gemstone (filter by gemstone_id)
3. Style (filter by style_id)
4. Price range (filter by min_price and max_price)

The implementation will:
1. Accept query parameters for each filter
2. Build a dynamic SQL query with WHERE clauses based on the provided parameters
3. Use parameterized queries to prevent SQL injection
4. Apply the filters in the database rather than in memory for better performance

This approach allows clients to request only the products that match specific criteria, making the API more efficient and useful.

## Implementing Sorting

Next, we'll add sorting capabilities to our endpoint. This will allow clients to specify the order in which products should be returned:

1. By name (alphabetical)
2. By price (low to high or high to low)
3. By metal (alphabetical)

The implementation will:
1. Accept sortBy and sortDirection query parameters
2. Add an ORDER BY clause to the SQL query based on these parameters
3. Provide a default sort order (by ID) if no sort parameter is specified

This sorting functionality gives clients control over how the data is organized in the response.

## Adding Pagination

To handle large result sets efficiently, we'll implement pagination. This prevents performance issues when the database contains many products and reduces the amount of data transferred over the network.

The pagination implementation will:
1. Accept page and pageSize parameters with default values
2. Use SQL's LIMIT and OFFSET clauses to retrieve only the requested page of results
3. Include a count of total matching products
4. Calculate pagination metadata (total pages, has next page, has previous page)

This approach helps clients navigate through large datasets efficiently without overwhelming the server or the client.

## Formatting the Response

The default response from our database query might include more data than needed or might not be structured in the most useful way. We'll enhance the response format to:

1. Include only the necessary product information
2. Include basic information about related entities (metal, gemstone, style)
3. Structure the response in a consistent and intuitive way
5. Include pagination metadata

This formatting makes the API response more concise and easier to consume by clients.

## Optimizing Database Queries

To improve performance, we'll optimize our database queries using several techniques:

1. Using SQL JOINs to retrieve related data in a single query
2. Selecting only the columns we need instead of using SELECT *
3. Adding appropriate WHERE clauses early in the query
4. Using indexes on commonly filtered columns
5. Implementing efficient pagination with LIMIT and OFFSET

These optimizations help ensure that our API remains performant even with large datasets and complex queries.

## Implementing the Enhanced Endpoint

Putting it all together, our enhanced endpoint will:

1. Accept multiple query parameters for filtering, sorting, and pagination
2. Build a dynamic SQL query based on these parameters
3. Execute the query efficiently using parameterized statements
4. Format the response with the appropriate structure and pagination metadata
5. Handle errors gracefully with appropriate status codes and messages

The implementation will use the DatabaseService to execute the SQL queries and map the results to our model objects.

## Conclusion

In this chapter, you've learned how to implement and enhance the endpoint for retrieving all products in the Jewelry Junction API. You've added filtering, sorting, pagination, and response formatting. You've also learned how to optimize database queries for better performance.

In the next chapter, we'll implement the endpoint for retrieving a single product by ID, including its related data.

## Practice Exercise

Enhance your Get All Products endpoint by:
1. Adding a filter for products by metal and gemstone combination (e.g., `?metalId=1&gemstoneId=2`)
2. Adding a filter for products by style and price range (e.g., `?styleId=3&minPrice=100&maxPrice=500`)
3. Adding a sort option for metal price (e.g., `?sortBy=metalPrice`)
4. Implementing a more advanced search that looks for keywords in product names and descriptions
5. Adding pagination to limit the number of results returned at once