# Update a Product

In this chapter, we'll implement the endpoint for updating a product in our Jewelry Junction API. This will involve validation, handling of related data, and ensuring data consistency.

## Learning Objectives

By the end of this chapter, you should be able to:
- Implement a PUT endpoint to update a product
- Validate the product data before updating
- Handle updates to related data
- Implement proper error handling
- Return appropriate responses

## Understanding the Product Update Process

Updating a product involves several considerations:
1. Validating that the product exists
2. Validating the updated data
3. Handling updates to related data (e.g., gemstones)
4. Ensuring data consistency
5. Returning the updated product

Let's implement an endpoint that addresses these considerations.

## Implementing the Basic Endpoint

For our product update endpoint, we'll implement a PUT request handler at the "/products/{id}" route. This basic implementation will:

1. Define a route handler for PUT /products/{id}
2. Extract the product ID from the URL path
3. Parse the request body to extract updated product information
4. Verify that the product exists
5. Update the product in the database
6. Return the updated product

The implementation will use the DatabaseService to execute the necessary SQL commands.

## Implementing Data Validation

Before updating a product, we need to validate the data to ensure it's complete and valid:

1. Verify that the product exists by querying the products table
2. Validate required fields (name, price, etc.)
3. Check that numeric values are valid (price > 0, stock quantity >= 0)
4. Verify that related entities (metal, category, discount) exist if referenced

This validation helps prevent invalid data from being stored in the database and provides clear error messages to clients.

## Handling Related Data

Products in our jewelry store have relationships with other entities, particularly gemstones. Updating a product might involve changing its associated gemstones, which requires special handling:

1. Begin a database transaction
2. Update the basic product information in the products table
3. Handle gemstone associations:
   - Delete existing associations from the product_gemstones table
   - Insert new associations based on the updated data
4. Commit the transaction if all operations succeed
5. Roll back the transaction if any operation fails

This approach ensures that related data is updated atomically with the product itself, maintaining data consistency.

## Using Transactions

To ensure data consistency when updating a product and its related data, we'll use a database transaction:

1. Begin a transaction using BEGIN TRANSACTION
2. Execute all SQL commands (UPDATE, DELETE, INSERT) within the transaction
3. Commit the transaction using COMMIT if all commands succeed
4. Roll back the transaction using ROLLBACK if any command fails

This approach prevents partial updates that could leave the database in an inconsistent state.

## Formatting the Response

After successfully updating a product, we'll format a comprehensive response that includes:

1. Basic product information (ID, name, description, price, etc.)
2. Calculated fields like discounted price
3. Related entity information (metal, category, discount)
4. Gemstone information with calculated values
5. A timestamp indicating when the update occurred

This detailed response provides clients with all the information they need about the updated product.

## Handling Errors

Our implementation will include robust error handling:

1. Use try-catch blocks to catch any exceptions during query execution
2. Return appropriate HTTP status codes:
   - 404 Not Found if the product doesn't exist
   - 400 Bad Request if the data is invalid
   - 500 Internal Server Error for unexpected errors
3. Include descriptive error messages in the response
4. Log detailed error information for debugging

This approach helps clients understand what went wrong and provides valuable information for debugging issues.

## Implementing Partial Updates

In many cases, clients may want to update only specific fields of a product, rather than providing all fields. To support this, we'll implement a PATCH endpoint:

1. Define a route handler for PATCH /products/{id}
2. Parse the request body to extract the fields to update
3. Build a dynamic SQL UPDATE statement that only updates the specified fields
4. Execute the statement and return the updated product

This approach is more flexible than the PUT endpoint, as it allows clients to update only the fields they need to change.

## Handling Concurrent Updates

To handle concurrent updates and prevent data loss, we can implement optimistic concurrency control:

1. Add a version or timestamp column to the products table
2. Include this value in responses and require it in update requests
3. Check that the version hasn't changed before applying updates
4. Return an error if the version has changed, indicating that someone else has updated the product

This approach helps prevent multiple users from overwriting each other's changes.

## Conclusion

In this chapter, you've learned how to implement endpoints for updating products in the Jewelry Junction API. You've explored different approaches to updates, including full updates and partial updates, and you've learned how to handle validation, related data, and response formatting.

In the next chapter, we'll implement the endpoint for joining related data, which will involve more complex queries and data manipulation.

## Practice Exercise

Enhance your product update functionality by:
1. Adding support for updating product images (URLs or file uploads)
2. Implementing versioning to track changes to products over time
3. Adding validation rules based on the product type (e.g., rings must have a size)
4. Implementing a feature to clone a product (create a new product based on an existing one)
5. Adding support for bulk updates (updating multiple products at once)