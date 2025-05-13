# Harbor Master: DELETE Docks and Haulers (Exercise)

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
});

// In HaulerEndpoints.cs
app.MapDelete("/haulers/{id}", async (int id, DatabaseService db) =>
{
    // TODO: Implement this endpoint
});
```

## Testing Your Implementation

1. Perform DELETE requests to `/docks{id}` and `/haulers{id}` — with valid `id` values and verify that you get a 204 response.
2. Perform DELETE requests to `/docks{id}` and `/haulers{id}` — with an invalid `id` value _(like 250)_ and verify that you get a 404 response.

## Conclusion

In this chapter, you've practiced implementing DELETE operations for docks and haulers in the Harbor Master API. You've applied the knowledge gained from the previous chapter to create endpoints for deleting different entity types and handled foreign key constraints.

Congratulations on completing the Harbor Master project! You've built a complete API that connects to a PostgreSQL database, handles various HTTP methods, and manages relationships between entities.

## Next Steps

[Deepen your learning by adding more features or connecting your client side project](./llm-guided-tasks.md)