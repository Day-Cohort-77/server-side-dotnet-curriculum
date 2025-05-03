# Get Single Product

In this chapter, we'll implement and enhance the endpoint for retrieving a single product by ID in our Jewelry Junction API. We'll explore how to include related data, handle errors, and format the response.

## Learning Objectives

By the end of this chapter, you should be able to:
- Implement a GET endpoint to retrieve a single product by ID
- Include related data in the response
- Handle cases where the product doesn't exist
- Format the response data
- Implement error handling

## Implementing the Basic Endpoint

We'll start by implementing a basic endpoint for retrieving a single product by ID. This endpoint will:

1. Define a route handler for GET /products/{id}
2. Extract the product ID from the URL path
3. Use the DatabaseService to execute a SQL query that retrieves the product with the specified ID
4. Return the product as a JSON response if found, or a 404 Not Found response if not found

The SQL query will select the product with the matching ID from the products table.

## Including Related Data

A product in our jewelry store has relationships with other entities like metals, categories, discounts, and gemstones. To provide a complete view of a product, we'll enhance our endpoint to include this related data:

1. Use SQL JOINs to retrieve data from multiple related tables in a single query
2. Join the products table with metals, categories, discounts, and product_gemstones tables
3. Further join product_gemstones with gemstones to get gemstone details
4. Join with reviews and customers tables to include product reviews

This approach allows us to retrieve all the necessary data in a single database query, rather than making multiple roundtrips.

## Handling Not Found Cases

When a client requests a product that doesn't exist, we need to return an appropriate response. We'll implement error handling to:

1. Check if the query returned any results
2. Return a 404 Not Found status code with a descriptive message if the product doesn't exist
3. Include enough information in the error response to help the client understand what went wrong

This approach provides clear feedback to clients when they request a resource that doesn't exist.

## Formatting the Response

To provide a well-structured and useful response, we'll format the product data:

1. Include basic product information (ID, name, description, price, etc.)
2. Add calculated fields like discounted price
3. Structure related data (metal, category, discount) as nested objects
4. Format gemstone information with calculated values
5. Include review information with customer details
6. Add summary fields like average rating and total gemstone value

This structured response makes it easier for clients to display product details without needing to make additional API calls or perform complex calculations.

## Implementing Error Handling

To make our API robust, we'll implement comprehensive error handling:

1. Use try-catch blocks to catch any exceptions that might occur during query execution
2. Log detailed error information for debugging
3. Return appropriate HTTP status codes based on the type of error
4. Include descriptive error messages in the response
5. Avoid exposing sensitive information in error responses

This approach helps with debugging issues and provides clear feedback to clients when something goes wrong.

## Implementing Related Endpoints

To provide more ways to access product information, we'll implement several related endpoints:

### Get Product Reviews

This endpoint will retrieve all reviews for a specific product:
1. Define a route handler for GET /products/{id}/reviews
2. Use a SQL query with JOINs to retrieve reviews and customer information
3. Format the response to include review details and customer information
4. Calculate and include the average rating

### Get Product Gemstones

This endpoint will retrieve all gemstones for a specific product:
1. Define a route handler for GET /products/{id}/gemstones
2. Use a SQL query with JOINs to retrieve gemstone details and carat information
3. Format the response to include gemstone details and calculated values
4. Calculate and include the total gemstone value

### Check Product Availability

This endpoint will provide availability information for a specific product:
1. Define a route handler for GET /products/{id}/availability
2. Use a SQL query to retrieve the product's stock quantity
3. Determine availability status based on stock quantity
4. Format the response to include availability information

Each of these endpoints provides focused information about a specific aspect of a product, making it easier for clients to get exactly what they need.

## Conclusion

In this chapter, you've learned how to implement and enhance the endpoint for retrieving a single product by ID in the Jewelry Junction API. You've included related data in the response, handled cases where the product doesn't exist, formatted the response data, and implemented error handling. You've also implemented related endpoints to provide more ways to access product information.

In the next chapter, we'll implement the endpoint for creating a new order, which will involve more complex operations like transactions and validation.

## Practice Exercise

Enhance your Get Single Product endpoint by:
1. Adding a parameter to control the inclusion of related data (e.g., `?includeReviews=true&includeGemstones=true`)
2. Adding a parameter to specify the number of reviews to include (e.g., `?reviewCount=5` for the 5 most recent reviews)
3. Adding a calculated field for the total value of the product (base price + gemstone value)
4. Adding a field that indicates whether the product is on sale (has an active discount)
5. Creating a new endpoint that returns similar products (products in the same category or with the same metal)