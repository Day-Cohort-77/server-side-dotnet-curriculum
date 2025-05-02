# Harbor Master: Implementing POST Operations for Haulers (Exercise)

In this chapter, you'll implement a POST endpoint to create a new hauler in the Harbor Master API. This is an exercise for you to practice what you've learned in the previous chapters.

## Learning Objectives

By the end of this chapter, you should be able to:
- Implement a data access method to insert a new hauler into the database
- Create a POST endpoint for haulers
- Validate input data
- Apply the knowledge gained from previous chapters to a new entity type

## Exercise Requirements

Your task is to implement the following:

1. Add a method to the `DatabaseService` class to:
   - Create a new hauler in the database

2. Create a new file called `Endpoints/HaulerEndpoints.cs` (if it doesn't exist already) with an extension method to:
   - Define a `POST /haulers` endpoint to create a new hauler

3. Update the `Program.cs` file to use your hauler endpoints

## Getting Started

Here's a step-by-step guide to help you get started:

### Step 1: Add Data Access Method

Open the `Services/DatabaseService.cs` file and add a method to create a new hauler. Think about:

- What SQL query do you need to write?
- What parameters do you need to include?
- How will you handle the generated ID?

### Step 2: Create Endpoint Extension Methods

Create a new file called `Endpoints/HaulerEndpoints.cs` (if it doesn't exist already) and define an extension method to map hauler endpoints. Consider:

- How will you validate the input data?
- What HTTP status codes should be returned in different scenarios?
- How will you document the endpoint for Swagger?

### Step 3: Update Program.cs

Update the `Program.cs` file to use your hauler endpoints. Remember to:

- Add the necessary using directive
- Call your extension method to map the hauler endpoints

## Hints

If you're stuck, here are some hints to help you:

1. The SQL query to create a new hauler would be:
   ```sql
   INSERT INTO haulers (name, capacity) VALUES (@name, @capacity) RETURNING id
   ```

2. Your `CreateHaulerAsync` method should return a `Hauler` object with the generated ID.

3. In your POST endpoint, you should validate that:
   - The name is not null or empty
   - The capacity is greater than zero

4. Your POST endpoint should return a 201 Created response with the location of the new resource.

## Example Implementation Structure

Here's a general structure for your implementation:

```csharp
// In DatabaseService.cs
public async Task<Hauler> CreateHaulerAsync(Hauler hauler)
{
    // TODO: Implement this method
}

// In HaulerEndpoints.cs
public static void MapHaulerEndpoints(this WebApplication app)
{
    // TODO: Add GET endpoints for haulers (similar to what you did for docks)

    // TODO: Add POST endpoint for haulers
    app.MapPost("/haulers", async (Hauler hauler, DatabaseService db) =>
    {
        // TODO: Validate input

        // TODO: Create the hauler

        // TODO: Return appropriate response
    })
    .WithName("CreateHauler")
    .WithOpenApi();
}

// In Program.cs
app.MapHaulerEndpoints();
```

## Testing Your Implementation

Once you've implemented the required functionality, run the application and test your endpoint:

1. Start the API:
   ```bash
   dotnet run
   ```

2. Open Swagger at `https://localhost:7042/swagger` (or the URL shown in your terminal)

3. Test the `POST /haulers` endpoint:
   - Click on the `POST /haulers` endpoint
   - Click the "Try it out" button
   - Enter a JSON request body:
     ```json
     {
       "name": "Coastal Shipping",
       "capacity": 12
     }
     ```
   - Click the "Execute" button
   - You should see a 201 Created response with the new hauler details, including the generated ID

4. Test the validation:
   - Try submitting with an empty name or a negative capacity
   - You should see appropriate error responses

## Conclusion

In this chapter, you've practiced implementing a POST endpoint for a new entity type. You've applied the knowledge gained from previous chapters to create a hauler creation endpoint in the Harbor Master API.

In the next chapter, we'll learn how to handle DELETE requests for ships.

## Next Steps

[Continue to the next chapter: Handling DELETE Requests for Ships](./harbor-master-delete-ships.md)