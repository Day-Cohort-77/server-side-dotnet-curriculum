# Harbor Master: Implementing DELETE Operations for Docks and Haulers (Exercise)

In this chapter, you'll implement DELETE endpoints to remove docks and haulers from the Harbor Master API. This is an exercise for you to practice what you've learned in the previous chapter.

## Learning Objectives

By the end of this chapter, you should be able to:
- Implement data access methods to delete docks and haulers
- Create DELETE endpoints for multiple entity types
- Handle foreign key constraints when deleting records
- Apply the knowledge gained from the previous chapter to new entity types

## Exercise Requirements

Your task is to implement the following:

1. Add methods to the `DatabaseService` class to:
   - Delete a dock by ID
   - Delete a hauler by ID

2. Add DELETE endpoints to:
   - `DockEndpoints.cs` for deleting docks
   - `HaulerEndpoints.cs` for deleting haulers

3. Handle the case where a dock or hauler has related ships

## Getting Started

Here's a step-by-step guide to help you get started:

### Step 1: Add Data Access Methods

Open the `Services/DatabaseService.cs` file and add methods to delete docks and haulers. Think about:

- What SQL queries do you need to write?
- How will you handle the case where a dock or hauler has related ships?
- How will you determine if a record was successfully deleted?

### Step 2: Add DELETE Endpoints

Add DELETE endpoints to your `DockEndpoints.cs` and `HaulerEndpoints.cs` files. Consider:

- What route parameters do you need?
- What HTTP status codes should be returned in different scenarios?
- How will you handle errors?

## Handling Foreign Key Constraints

Remember that in our database schema, we set up the foreign key constraints with `ON DELETE SET NULL`. This means that when a dock or hauler is deleted, any references to it in the ships table will be set to NULL rather than causing an error or preventing the delete.

However, you might want to add additional validation or business logic. For example:

1. You might want to prevent deleting a dock if it has ships assigned to it
2. You might want to return information about affected ships when deleting a dock or hauler

Think about how you would implement these requirements.

## Hints

If you're stuck, here are some hints to help you:

1. The SQL query to delete a dock would be:
   ```sql
   DELETE FROM docks WHERE id = @id
   ```

2. The SQL query to delete a hauler would be:
   ```sql
   DELETE FROM haulers WHERE id = @id
   ```

3. To check if a dock has ships before deleting it, you could use:
   ```sql
   SELECT COUNT(*) FROM ships WHERE dock_id = @id
   ```

4. Your `DeleteDockAsync` and `DeleteHaulerAsync` methods should return a boolean indicating whether a record was deleted.

5. Your DELETE endpoints should return:
   - 204 No Content if the record was deleted
   - 404 Not Found if the record wasn't found
   - 400 Bad Request if the record can't be deleted due to business rules (optional)

## Example Implementation Structure

Here's a general structure for your implementation:

```csharp
// In DatabaseService.cs
public async Task<bool> DeleteDockAsync(int id)
{
    // TODO: Implement this method
}

public async Task<bool> DeleteHaulerAsync(int id)
{
    // TODO: Implement this method
}

// In DockEndpoints.cs
app.MapDelete("/docks/{id}", async (int id, DatabaseService db) =>
{
    // TODO: Implement this endpoint
})
.WithName("DeleteDock")
.WithOpenApi();

// In HaulerEndpoints.cs
app.MapDelete("/haulers/{id}", async (int id, DatabaseService db) =>
{
    // TODO: Implement this endpoint
})
.WithName("DeleteHauler")
.WithOpenApi();
```

## Advanced Challenge: Implementing Cascading Deletes

As an advanced challenge, you could implement cascading deletes for docks and haulers. This would mean that when a dock or hauler is deleted, all related ships would also be deleted.

To do this, you would:

1. Change the foreign key constraints in the database schema:
   ```sql
   FOREIGN KEY (dock_id) REFERENCES docks(id) ON DELETE CASCADE,
   FOREIGN KEY (hauler_id) REFERENCES haulers(id) ON DELETE CASCADE
   ```

2. Or handle the cascading delete in your code using transactions:
   ```csharp
   public async Task<bool> DeleteDockWithShipsAsync(int id)
   {
       using var connection = CreateConnection();
       await connection.OpenAsync();

       // Begin a transaction
       using var transaction = await connection.BeginTransactionAsync();

       try
       {
           // Delete related ships first
           using var deleteShipsCommand = new NpgsqlCommand(
               "DELETE FROM ships WHERE dock_id = @id",
               connection, transaction as NpgsqlTransaction);
           deleteShipsCommand.Parameters.AddWithValue("@id", id);
           await deleteShipsCommand.ExecuteNonQueryAsync();

           // Then delete the dock
           using var deleteDockCommand = new NpgsqlCommand(
               "DELETE FROM docks WHERE id = @id",
               connection, transaction as NpgsqlTransaction);
           deleteDockCommand.Parameters.AddWithValue("@id", id);
           int rowsAffected = await deleteDockCommand.ExecuteNonQueryAsync();

           // Commit the transaction
           await transaction.CommitAsync();

           return rowsAffected > 0;
       }
       catch
       {
           // Rollback the transaction if an error occurs
           await transaction.RollbackAsync();
           throw;
       }
   }
   ```

## Testing Your Implementation

Once you've implemented the required functionality, run the application and test your endpoints:

1. Start the API:
   ```bash
   dotnet run
   ```

2. Open Swagger at `https://localhost:7042/swagger` (or the URL shown in your terminal)

3. Test the `DELETE /docks/{id}` endpoint:
   - First, get a list of docks to find an ID to delete
   - Click on the `DELETE /docks/{id}` endpoint
   - Click the "Try it out" button
   - Enter the ID of the dock you want to delete
   - Click the "Execute" button
   - You should see a 204 No Content response if the dock was deleted

4. Test the `DELETE /haulers/{id}` endpoint:
   - First, get a list of haulers to find an ID to delete
   - Click on the `DELETE /haulers/{id}` endpoint
   - Click the "Try it out" button
   - Enter the ID of the hauler you want to delete
   - Click the "Execute" button
   - You should see a 204 No Content response if the hauler was deleted

5. Verify the records were deleted:
   - Use the `GET /docks` and `GET /haulers` endpoints to see that the deleted records are no longer in the lists

## Conclusion

In this chapter, you've practiced implementing DELETE operations for docks and haulers in the Harbor Master API. You've applied the knowledge gained from the previous chapter to create endpoints for deleting different entity types and handled foreign key constraints.

Congratulations on completing the Harbor Master project! You've built a complete API that connects to a PostgreSQL database, handles various HTTP methods, and manages relationships between entities.

## Next Steps

Now that you've completed the Harbor Master project, you can:

1. Add more features to the API, such as:
   - Filtering and sorting for GET endpoints
   - Pagination for large result sets
   - Authentication and authorization
   - Logging and error handling middleware

2. Build a client application to consume the API, such as:
   - A web application using HTML, CSS, and JavaScript
   - A mobile application using a framework like React Native
   - A desktop application using a framework like Electron

3. Explore other aspects of ASP.NET Core, such as:
   - MVC architecture
   - Razor Pages
   - Blazor
   - SignalR for real-time communication