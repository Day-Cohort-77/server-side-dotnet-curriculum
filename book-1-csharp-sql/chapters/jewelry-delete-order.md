# Delete an Order

In this chapter, we'll implement the endpoint for deleting an order in our Jewelry Junction API. This will involve handling transactions, restoring product stock quantities, and implementing proper error handling.

## Learning Objectives

By the end of this chapter, you should be able to:
- Implement a DELETE endpoint to remove an order
- Handle errors and edge cases
- Implement proper validation before deletion
- Return appropriate responses

## Understanding the Order Deletion Process

Deleting an order involves several steps:
1. Validating that the order exists and can be deleted
2. Removing order items associated with the order
3. Removing the order itself

## Implementing the Endpoint

For our order deletion endpoint, we'll implement a DELETE request handler at the "/orders/{id}" route. This endpoint will:

1. Define a route handler for DELETE /orders/{id}
2. Extract the order ID from the URL path
3. Retrieve the order and its items
4. Validate that the order exists and can be deleted
5. Delete order items
6. Delete the order
7. Return a success response

The implementation will use the DatabaseService to execute the necessary SQL commands within a transaction.

## Validating the Order

Before deleting an order, we need to validate that it exists and can be deleted:

1. Query the database to check if an order with the specified ID exists
2. Verify that the order is in a state that allows deletion (e.g., only recent orders can be deleted)
3. Return appropriate error responses if validation fails

This validation prevents the deletion of orders that don't exist or shouldn't be deleted.

## Deleting Order Items

Before deleting the order itself, we need to delete all associated order items:

1. Retrieve all order items associated with the order
2. Delete each order item from the database

This ensures that we don't leave orphaned order items in the database.

## Deleting the Order

After deleting all associated order items, we can delete the order itself:

1. Execute a DELETE SQL command to remove the order from the database
2. Verify that the deletion was successful
3. Return a success response to the client

This completes the order deletion process.

## Conclusion

In this chapter, you've learned how to implement endpoints for deleting orders in the Jewelry Junction API. You've explored different approaches to deletion, including hard delete and soft delete, and you've learned how to handle transactions, restore product stock quantities, and implement proper error handling. You've also learned how to implement a restore endpoint for soft-deleted orders.

In the next chapter, we'll implement the endpoint for updating a product, which will involve validation and handling of related data.

## Optional Practice Exercises

### Implementing Soft Delete

In many real-world applications, it's preferable to implement "soft delete" rather than actually removing records from the database. This involves marking records as deleted rather than physically removing them, which allows for data recovery and audit trails.

To implement soft delete for orders:

1. Add an `IsDeleted` column to the orders table
2. Add a `DeletedAt` timestamp column to track when the order was deleted
3. Instead of deleting the order, update these columns:
   - Set `IsDeleted` to true
   - Set `DeletedAt` to the current timestamp
   - Change the order status to "Cancelled"
4. Update all queries to filter out deleted orders (WHERE `IsDeleted` = false)

This approach preserves the order history while still allowing for "deletion" from the user's perspective.

### Implementing a Restore Endpoint

To complement the soft delete functionality, we can implement an endpoint to restore deleted orders:

1. Define a route handler for POST `/orders/{id}/restore`
2. Verify that the order exists and is marked as deleted
3. Check if there's sufficient stock available for all items in the order
4. Begin a transaction
5. Update the order to clear the `IsDeleted` flag and `DeletedAt` timestamp
6. Restore the order status to its original value
7. Deduct product stock quantities again
8. Commit the transaction

This restoration process essentially reverses the soft delete operation, but with an important check to ensure there's still sufficient inventory available.


### Additional Optional Features

1. Adding support for partial order cancellation (cancelling specific items in an order)
2. Implementing deletion rules based on the order date _(i.e., recent orders can be deleted, older orders cannot. You can pick any timespan for your implementation)_