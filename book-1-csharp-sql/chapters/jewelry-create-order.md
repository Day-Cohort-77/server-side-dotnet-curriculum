# Create New Order

In this chapter, we'll implement the endpoint for creating a new order in our Jewelry Junction API. This will involve more complex operations like transactions, validation, and updating related data.

## Learning Objectives

By the end of this chapter, you should be able to:
- Implement a POST endpoint to create a new order
- Validate the order data before processing
- Create order records in the database
- Handle errors and edge cases
- Return appropriate responses

## Understanding the Order Creation Process

Creating an order involves several steps:
1. Validating the order data (products exist, etc.)
2. Creating the order record
3. Creating order item records for each product in the order
4. Calculating the total order amount
5. Returning the created order with its items

## Implementing the Endpoint

For our order creation endpoint, we'll implement a POST request handler at the "/orders" route. This endpoint will:

1. Define a route handler for POST /orders
2. Parse the request body to extract order information
3. Validate the order data
4. Create the order and order items
5. Calculate the total order amount
6. Return the created order with its details

The implementation will use the DatabaseService to execute the necessary SQL commands within a transaction.

## Validating Order Data

Before processing an order, we need to validate the data to ensure it's complete and valid:

1. Check that the order contains at least one item
2. Verify that each product exists by querying the products table
3. Validate that quantities are positive

This validation helps prevent invalid orders from being processed and provides clear error messages to clients.

## Creating the Order and Order Items

The core of our implementation will:

1. Insert a new record into the orders table with order date and initial status
2. Retrieve the generated order ID
3. For each item in the order:
   - Insert a record into the order_items table with product ID, quantity, and unit price

## Creating the Order and Order Items

These operations will be performed using parameterized SQL commands to prevent SQL injection.

## Calculating the Total Amount

To calculate the total order amount:

1. Sum the (unit price × quantity) for each order item
2. Update the total_amount column in the orders table with this sum

This calculation ensures that the order total accurately reflects the prices at the time of purchase, including any applicable discounts.

## Handling Edge Cases

Our implementation will handle various edge cases:

1. Invalid product IDs
2. Empty order (no items)
3. Database errors

For each case, we'll provide appropriate error responses with clear messages to help clients understand what went wrong.

## Formatting the Response

After successfully creating an order, we'll format a comprehensive response that includes:

1. Basic order information (ID, date, status, total amount)
2. Detailed information about each order item:
   - Product details (name, metal, gemstone, style)
   - Pricing information (unit price)
   - Quantity and subtotal
4. Summary information (item count, creation timestamp)

This detailed response provides clients with all the information they need to display an order confirmation.

## Implementing Request Validation

To make our API more robust, we'll implement request validation:

1. Define a clear request structure that clients must follow
2. Validate that all required fields are present
3. Check that data types are correct
4. Validate business rules (e.g., quantities must be positive)

This validation happens before any database operations, preventing invalid requests from affecting the database.

## Conclusion

In this chapter, you've learned how to implement the endpoint for creating a new order in the Jewelry Junction API. You've used transactions to ensure data consistency, validated the order data before processing, updated related data, and handled errors and edge cases. You've also learned how to structure the response to provide useful information to clients.

In the next chapter, we'll implement the endpoint for deleting an order, which will involve similar concepts like transactions and validation.

## Practice Exercise

Enhance your Create Order endpoint by:
1. Adding validation for the quantity (must be positive)
2. Adding support for shipping information (address, shipping method, etc.)
3. Adding support for payment information (payment method, transaction ID, etc.)
4. Implementing a feature to calculate shipping costs based on order total
5. Adding the ability to create an order with products of specific styles only