# Harbor Master: Implementing GET Operations for Docks (Exercise)

In this chapter, you'll implement GET operations to retrieve dock data from the PostgreSQL database. This is an exercise for you to practice what you've learned in the previous chapter.

## Learning Objectives

By the end of this chapter, you should be able to:
- Implement data access methods for a new entity type
- Create GET endpoints for retrieving dock data
- Apply the knowledge gained from the previous chapter to a new scenario

## Exercise Requirements

Your task is to implement the following:

1. Add methods to the `DatabaseService` class to:
   - Retrieve all docks from the database
   - Retrieve a single dock by ID

2. Create a new file called `Endpoints/DockEndpoints.cs` with an extension method to:
   - Define a `GET /docks` endpoint to get all docks
   - Define a `GET /docks/{id}` endpoint to get a dock by ID

3. Update the `Program.cs` file to use your dock endpoints

## Getting Started

Here's a step-by-step guide to help you get started:

### Step 1: Add Data Access Methods

Open the `Services/DatabaseService.cs` file and add methods to retrieve dock data. Think about:

- What SQL queries do you need to write?
- How will you map the database results to the `Dock` model?
- How will you handle the case when a dock with a given ID doesn't exist?

### Step 2: Create Endpoint Extension Methods

Create a new file called `Endpoints/DockEndpoints.cs` and define an extension method to map dock endpoints. Consider:

- How will you structure the endpoints?
- What HTTP status codes should be returned in different scenarios?
- How will you document the endpoints for Swagger?

### Step 3: Update Program.cs

Update the `Program.cs` file to use your dock endpoints. Remember to:

- Add the necessary using directive
- Call your extension method to map the dock endpoints

## Hints

If you're stuck, here are some hints to help you:

1. The SQL query to get all docks would be:
   ```sql
   SELECT id, location, capacity FROM docks
   ```

2. The SQL query to get a dock by ID would be:
   ```sql
   SELECT id, location, capacity FROM docks WHERE id = @id
   ```

3. Your `GetAllDocksAsync` method should return a `List<Dock>`.

4. Your `GetDockByIdAsync` method should return a `Dock?` (nullable Dock).

5. In your endpoint for getting a dock by ID, you should return `Results.NotFound()` if the dock doesn't exist.

## Testing Your Implementation

Once you've implemented the required functionality, run the application and test your endpoints:

1. Restart your debugger

2. Test the `GET /docks` endpoint: You should see a response with the list of docks

3. Test the `GET /docks/{id}` endpoint: You should see a response with the dock details

4. Try the `GET /docks/{id}` endpoint with an invalid ID: You should see a 404 Not Found response

## Conclusion

In this chapter, you've practiced implementing GET operations for a new entity type. You've applied the knowledge gained from the previous chapter to create endpoints for retrieving dock data from the database.

In the next chapter, we'll learn how to handle POST requests to create a dock.

## Next Steps

[Continue to the next chapter: Handling POST Requests to Create a Dock](./harbor-master-post-docks.md)