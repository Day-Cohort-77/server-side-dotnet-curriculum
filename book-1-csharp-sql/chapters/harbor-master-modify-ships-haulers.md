# Harbor Master: Modify Ships and Haulers (Exercise)

In this chapter, you'll implement PUT endpoints to modify ships and haulers in the Harbor Master API. This is an exercise for you to practice what you've learned in the previous chapter.

## Learning Objectives

By the end of this chapter, you should be able to:
- Implement data access methods to update ships and haulers
- Create PUT endpoints for multiple entity types
- Validate input data for updates
- Apply business rules to ensure data integrity
- Apply the knowledge gained from the previous chapter to new entity types

## Exercise Requirements

Your task is to implement the following:

1. Add methods to the `DatabaseService` class to:
   - Update a ship by ID
   - Update a hauler by ID

2. Add PUT endpoints to:
   - `ShipEndpoints.cs` for updating ships
   - `HaulerEndpoints.cs` for updating haulers

3. Implement appropriate validation and business rules:
   - For ships: Ensure the dock exists and has capacity
   - For haulers: Ensure the capacity is sufficient for current assignments

## Getting Started

Here's a step-by-step guide to help you get started:

### Step 1: Add Data Access Methods

Open the `Services/DatabaseService.cs` file and add methods to update ships and haulers. Think about:

- What SQL queries do you need to write?
- What parameters do you need to include?
- How will you handle validation of foreign keys?
- How will you return the updated entity?

### Step 2: Add PUT Endpoints

Add PUT endpoints to your `ShipEndpoints.cs` and `HaulerEndpoints.cs` files. Consider:

- What route parameters do you need?
- What validation should you perform on the input data?
- What business rules should you enforce?
- What HTTP status codes should be returned in different scenarios?

### Step 3: Implement Business Rules

For ships:
- Ensure the dock exists before assigning a ship to it
- Check that the dock has enough capacity for the ship
- If the ship is already assigned to a dock, handle the capacity changes correctly

For haulers:
- Ensure the capacity is sufficient for the number of ships currently assigned to the hauler
- Validate that the name is not empty and the capacity is greater than zero

## Hints

If you're stuck, here are some hints to help you:

1. The SQL query to update a ship would be:
   ```sql
   UPDATE ships
   SET name = @name,
       dock_id = @dockId,
       hauler_id = @haulerId
   WHERE id = @id
   ```

2. The SQL query to update a hauler would be:
   ```sql
   UPDATE haulers
   SET name = @name,
       capacity = @capacity
   WHERE id = @id
   ```

3. To check if a hauler has enough capacity for its current ships:
   ```sql
   SELECT COUNT(*)
   FROM ships
   WHERE hauler_id = @haulerId
   ```

4. Your `UpdateShipAsync` and `UpdateHaulerAsync` methods should return the updated entity.

5. Your PUT endpoints should:
   - Check if the entity exists
   - Validate the input data
   - Enforce business rules
   - Update the entity
   - Return the updated entity

## Example Implementation Structure

Here's a general structure for your implementation:

### For Ships

```csharp
// In DatabaseService.cs
public async Task<Ship> UpdateShipAsync(Ship ship)
{
    // TODO: Implement this method
}

// In ShipEndpoints.cs
app.MapPut("/ships/{id}", async (int id, Ship updatedShip, DatabaseService db) =>
{
    try
    {
        // Check if the ship exists
        var existingShip = await db.GetShipByIdAsync(id);
        if (existingShip == null)
        {
            return Results.NotFound($"Ship with ID {id} not found");
        }

        // Validate input
        if (string.IsNullOrWhiteSpace(updatedShip.Name))
        {
            return Results.BadRequest("Name is required");
        }

        // Check if the dock exists and has capacity
        if (updatedShip.DockId.HasValue)
        {
            var dock = await db.GetDockByIdAsync(updatedShip.DockId.Value);
            if (dock == null)
            {
                return Results.BadRequest($"Dock with ID {updatedShip.DockId.Value} not found");
            }

            // TODO: Check dock capacity
        }

        // Set the ID from the route parameter
        updatedShip.Id = id;

        // Update the ship
        var result = await db.UpdateShipAsync(updatedShip);

        // Return the updated ship
        return Results.Ok(result);
    }
    catch (Exception ex)
    {
        return Results.Problem($"An error occurred while updating the ship: {ex.Message}");
    }
});
```

## Testing Your Implementation

Once you've implemented the required functionality, run the application and test your endpoints:

1. Restart your debugger

2. Test the `PUT /ships/{id}` endpoint:
   - Choose an existing ship ID
   - Enter a JSON request body:
     ```json
     {
       "name": "Updated Cargo Ship",
       "dockId": 1,
       "haulerId": 2
     }
     ```
   - You should see a 200 OK response with the updated ship details

3. Test the `PUT /haulers/{id}` endpoint:
   - Choose an existing hauler ID
   - Enter a JSON request body:
     ```json
     {
       "name": "Updated Shipping Co",
       "capacity": 20
     }
     ```
   - You should see a 200 OK response with the updated hauler details

4. Test the validation and business rules:
   - Try updating a ship with an empty name
   - Try updating a ship with a non-existent dock ID
   - Try updating a hauler with a capacity less than the number of ships assigned to it

## Conclusion

In this chapter, you've practiced implementing PUT operations for ships and haulers in the Harbor Master API. You've applied the knowledge gained from the previous chapter to create endpoints for updating different entity types and enforced business rules to ensure data integrity.

Congratulations on completing the Harbor Master project! You've built a complete API that connects to a PostgreSQL database, handles various HTTP methods (GET, POST, PUT, DELETE), and manages relationships between entities.

## Next Steps

[Deepen your learning by adding more features or connecting your client side project](./llm-guided-tasks.md)