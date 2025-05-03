# Get Single Product

In this chapter, we'll implement the endpoint for retrieving a single product by ID in our Jewelry Junction API. We'll explore how to include related data, handle errors, and format the response.

## Learning Objectives

By the end of this chapter, you should be able to:
- Implement a GET endpoint to retrieve a single product by ID
- Include related data in the response
- Handle cases where the product doesn't exist
- Format the response data
- Implement error handling

## Implementing the Basic Endpoint

We'll start by implementing a basic endpoint in the `ProductEndpoints.cs` module for retrieving a single product by ID. This endpoint will:

1. Define a route handler for GET `/products/{id}`
2. Extract the product ID from the URL path
3. Use the **DatabaseService** to execute a SQL query that retrieves the product with the specified ID
4. Return the product as a JSON response if found, or a 404 Not Found response if not found

The SQL query will select the product with the matching ID from the products table.

## Product Details SQL JOIN

You need to create a new `GetProductByIdAsync` task in your **DatabaseService** module. Your SQL for this task must utilize the powerful `JOIN` operation in SQL to provide more details to the response body.

Use the following SQL in the **NpgsqlCommand** for this task.
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
LEFT JOIN metals m ON p.metal_id = m.id
LEFT JOIN gemstones g ON p.gemstone_id = g.id
LEFT JOIN styles s ON p.style_id = s.id
WHERE p.id = @productId;
```

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
2. Structure related data (metal, gemstone, style) as nested objects
3. Add calculated fields like total value (based on metal price per gram and gemstone price per carat)

This structured response makes it easier for clients to display product details without needing to make additional API calls or perform complex calculations.

## Implementing Error Handling

To make our API robust, we'll implement comprehensive error handling:

1. Use try-catch blocks to catch any exceptions that might occur during query execution
2. Log detailed error information for debugging
3. Return appropriate HTTP status codes based on the type of error
4. Include descriptive error messages in the response
5. Avoid exposing sensitive information in error responses

This approach helps with debugging issues and provides clear feedback to clients when something goes wrong.

## Conclusion

In this chapter, you've learned how to implement and enhance the endpoint for retrieving a single product by ID in the Jewelry Junction API. You've included related data in the response, handled cases where the product doesn't exist, formatted the response data, and implemented error handling.

In the next chapter, we'll implement the endpoint for creating a new order, which will involve more complex operations like transactions and validation.

## Optional Practice Exercises

Enhance your Get Single Product endpoint by:

1. Adding a parameter to control the inclusion of related data (e.g., `?includeMetal=true&includeGemstone=true&includeStyle=true`)
2. Adding a calculated field for the total value of the product (base price + metal value + gemstone value)
3. Creating a new endpoint that returns similar products (products with the same style or with the same metal)
4. Adding an endpoint to get all products with a specific metal and gemstone combination
5. Implementing an endpoint to get products sorted by price within a specific style