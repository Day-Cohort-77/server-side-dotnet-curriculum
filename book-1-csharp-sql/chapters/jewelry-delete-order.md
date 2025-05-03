# Delete an Order

In this chapter, we'll implement the endpoint for deleting an order in our Jewelry Junction API. This will involve handling transactions, restoring product stock quantities, and implementing proper error handling.

## Learning Objectives

By the end of this chapter, you should be able to:
- Implement a DELETE endpoint to remove an order
- Use transactions to ensure data consistency
- Restore product stock quantities when an order is deleted
- Handle errors and edge cases
- Implement proper validation before deletion

## Understanding the Order Deletion Process

Deleting an order is not as simple as removing a record from the database. It involves several steps:
1. Validating that the order exists and can be deleted
2. Restoring product stock quantities for all items in the order
3. Removing order items associated with the order
4. Removing the order itself

Since these operations need to be atomic (all succeed or all fail), we'll use a transaction to ensure data consistency.

## Implementing the Endpoint

For our order deletion endpoint, we'll implement a DELETE request handler at the "/orders/{id}" route. This endpoint will:

1. Define a route handler for DELETE /orders/{id}
2. Extract the order ID from the URL path
3. Begin a database transaction
4. Retrieve the order and its items
5. Validate that the order exists and can be deleted
6. Restore product stock quantities
7. Delete order items
8. Delete the order
9. Commit the transaction if all operations succeed
10. Return a success response
11. Roll back the transaction if any operation fails

The implementation will use the DatabaseService to execute the necessary SQL commands within a transaction.

## Validating the Order

Before deleting an order, we need to validate that it exists and can be deleted:

1. Query the database to check if an order with the specified ID exists
2. Verify that the order is in a state that allows deletion (e.g., only pending orders can be deleted)
3. Return appropriate error responses if validation fails

This validation prevents the deletion of orders that don't exist or shouldn't be deleted.

## Restoring Product Stock Quantities

When an order is deleted, we need to restore the stock quantities of the products that were part of the order:

1. Retrieve all order items associated with the order
2. For each order item:
   - Retrieve the product
   - Increase the product's stock quantity by the order item quantity
   - Update the product record in the database

This ensures that the inventory is correctly adjusted when an order is deleted.

## Using Transactions

To ensure data consistency, we'll use a database transaction that encompasses all the operations involved in deleting an order:

1. Begin a transaction using BEGIN TRANSACTION
2. Execute all SQL commands (SELECT, UPDATE, DELETE) within the transaction
3. Commit the transaction using COMMIT if all commands succeed
4. Roll back the transaction using ROLLBACK if any command fails

This approach ensures that either all parts of the order deletion succeed, or none of them do, preventing partial updates that could leave the database in an inconsistent state.

## Enhancing the Response

To provide more information about the deleted order, we'll enhance our response:

1. Retrieve detailed information about the order and its items before deletion
2. Format a response that includes:
   - Basic order information (ID, date, status, total amount)
   - Customer information
   - Order item details
   - Summary information (item count)

This detailed response helps clients understand what was deleted and can be useful for displaying confirmation messages or maintaining audit trails.

## Implementing Soft Delete

In many real-world applications, it's preferable to implement "soft delete" rather than actually removing records from the database. This involves marking records as deleted rather than physically removing them, which allows for data recovery and audit trails.

To implement soft delete for orders:

1. Add an is_deleted column to the orders table
2. Add a deleted_at timestamp column to track when the order was deleted
3. Instead of deleting the order, update these columns:
   - Set is_deleted to true
   - Set deleted_at to the current timestamp
   - Change the order status to "Cancelled"
4. Update all queries to filter out deleted orders (WHERE is_deleted = false)

This approach preserves the order history while still allowing for "deletion" from the user's perspective.

## Implementing Hard Delete with Confirmation

For cases where permanent deletion is necessary, we can implement a separate endpoint that requires confirmation:

1. Define a route handler for DELETE /orders/{id}/hard
2. Require a confirmation parameter with a specific value
3. Perform the same validation and transaction handling as the soft delete endpoint
4. Physically delete the order items and order from the database

This approach helps prevent accidental permanent deletions by requiring explicit confirmation.

## Implementing a Restore Endpoint

To complement the soft delete functionality, we can implement an endpoint to restore deleted orders:

1. Define a route handler for POST /orders/{id}/restore
2. Verify that the order exists and is marked as deleted
3. Check if there's sufficient stock available for all items in the order
4. Begin a transaction
5. Update the order to clear the is_deleted flag and deleted_at timestamp
6. Restore the order status to its original value
7. Deduct product stock quantities again
8. Commit the transaction

This restoration process essentially reverses the soft delete operation, but with an important check to ensure there's still sufficient inventory available.

## Conclusion

In this chapter, you've learned how to implement endpoints for deleting orders in the Jewelry Junction API. You've explored different approaches to deletion, including hard delete and soft delete, and you've learned how to handle transactions, restore product stock quantities, and implement proper error handling. You've also learned how to implement a restore endpoint for soft-deleted orders.

In the next chapter, we'll implement the endpoint for updating a product, which will involve validation and handling of related data.

## Practice Exercise

Enhance your order deletion functionality by:
1. Adding an audit trail for order deletions (who deleted it, when, why)
2. Implementing a "recycle bin" feature that allows viewing and restoring deleted orders
3. Adding a scheduled task to permanently delete orders that have been soft-deleted for more than 30 days
4. Adding support for partial order cancellation (cancelling specific items in an order)
5. Implementing different deletion rules based on the order status (e.g., pending orders can be deleted, shipped orders can only be marked as returned)