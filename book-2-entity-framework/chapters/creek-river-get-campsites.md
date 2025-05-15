# Creating the first endpoint that will GET campsites

In this chapter, we'll create our first API endpoint using Entity Framework Core to retrieve campsite data from our database. We'll implement two endpoints: one to get all campsites and another to get a specific campsite by its ID.

## Understanding API Endpoints with EF Core

In our organized approach, we define endpoints in separate classes by resource type. This keeps our code clean and maintainable. For campsite endpoints, we'll use the `CampsiteEndpoints.cs` file we created in the previous chapter.

## Getting All Campsites

Let's start by implementing the endpoint to get all campsites. This endpoint is already defined in our `CampsiteEndpoints.cs` file:

```csharp
// GET /api/campsites - Get all campsites
app.MapGet("/api/campsites", (CreekRiverDbContext db) =>
{
    return db.Campsites.ToList();
});
```

Let's break down this code:

- `app.MapGet("/api/campsites", ...)`: This maps HTTP GET requests to the `/api/campsites` URL to our handler function.

- `(CreekRiverDbContext db) => { ... }`: This is the handler function. The `CreekRiverDbContext` parameter is injected by ASP.NET Core's dependency injection system.

- `db.Campsites`: This accesses the `Campsites` DbSet in our DbContext, which represents the `Campsites` table in the database.

- `db.Campsites.ToList()`: This executes the query and returns all campsites as a list.

## Understanding LINQ and Entity Framework

The query in our endpoint uses LINQ (Language Integrated Query), which is a set of features in C# that provides a consistent syntax for querying different types of data sources.

When used with Entity Framework Core, LINQ queries are translated into SQL queries that are executed against the database. For example, the query in our endpoint might be translated to something like:

```sql
SELECT * FROM "Campsites"
```

This is a powerful feature of Entity Framework Core: it allows you to write queries in C# that are translated to SQL at runtime.

## Getting a Specific Campsite by ID

Now, let's look at the endpoint to get a specific campsite by its ID:

```csharp
// GET /api/campsites/{id} - Get a specific campsite by ID
app.MapGet("/api/campsites/{id}", (CreekRiverDbContext db, int id) =>
{
    var campsite = db.Campsites
        .Include(c => c.CampsiteType)
        .SingleOrDefault(c => c.Id == id);

    if (campsite == null)
    {
        return Results.NotFound();
    }

    return Results.Ok(campsite);
});
```

This endpoint is similar to the previous one, but with a few key differences:

- `"/api/campsites/{id}"`: This URL pattern includes a parameter `{id}` that captures the ID from the URL.

- `(CreekRiverDbContext db, int id) => { ... }`: The handler function now takes an additional `id` parameter, which is bound to the `{id}` parameter in the URL.

- `.Include(c => c.CampsiteType)`: This tells EF Core to include the related `CampsiteType` entity in the query. This is similar to a JOIN in SQL.

- `.SingleOrDefault(c => c.Id == id)`: This filters the results to the campsite with the specified ID. `SingleOrDefault` returns null if no matching campsite is found.

- Error handling: We check if the campsite is null and return a 404 Not Found response if it is.

## Testing the Endpoints

Now that we've created our endpoints, let's test them:

1. Run the application with `dotnet run` or by pressing F5 in Visual Studio.

2. Open a web browser or a tool like Yaak and navigate to:
   - `https://localhost:<port>/api/campsites` to get all campsites
   - `https://localhost:<port>/api/campsites/1` to get the campsite with ID 1

3. You should see JSON responses with the campsite data.

## Understanding the Results

When you call the `/api/campsites` endpoint, you should see a JSON array of campsite objects:

```json
[
  {
    "id": 1,
    "nickname": "Barred Owl",
    "imageUrl": "https://tnstateparks.com/assets/images/content-images/campgrounds/249/colsp-area2-site73.jpg",
    "campsiteTypeId": 1,
    "campsiteType": null,
    "reservations": []
  },
  {
    "id": 2,
    "nickname": "Red Fox",
    "imageUrl": "https://tnstateparks.com/assets/images/content-images/campgrounds/249/colsp-area2-site73.jpg",
    "campsiteTypeId": 2,
    "campsiteType": null,
    "reservations": []
  },
  // More campsites...
]
```

When you call the `/api/campsites/{id}` endpoint, you should see a single campsite object with its related campsite type:

```json
{
  "id": 1,
  "nickname": "Barred Owl",
  "imageUrl": "https://tnstateparks.com/assets/images/content-images/campgrounds/249/colsp-area2-site73.jpg",
  "campsiteTypeId": 1,
  "campsiteType": {
    "id": 1,
    "campsiteTypeName": "Tent",
    "maxReservationDays": 7,
    "feePerNight": 15.99
  },
  "reservations": []
}
```

## Benefits of Organized Endpoints

By organizing our endpoints in separate files by resource, we gain several benefits:

1. **Better organization** - Endpoints are grouped by resource, making it easier to find and modify them.
2. **Improved maintainability** - Each resource's endpoints are contained in a single file, reducing the complexity of `Program.cs`.
3. **Scalability** - As your application grows, you can add new endpoint classes without cluttering `Program.cs`.

## Conclusion

In this chapter, we've created our first API endpoints using Entity Framework Core to retrieve campsite data from our database. We've learned how to:

1. Organize endpoints by resource in separate files
2. Create an endpoint to get all campsites
3. Create an endpoint to get a specific campsite by ID
4. Include related data in our queries
5. Handle errors and return appropriate HTTP responses

These endpoints demonstrate the power of Entity Framework Core: we can write simple, expressive queries in C# that are translated to SQL at runtime, and we can easily include related data in our queries.

In the next chapter, we'll create an endpoint to add a new campsite to the database.

Up Next: [Chapter to POST a campsite](./creek-river-post-campsite.md)

## 🔍 Additional Materials

- [ASP.NET Core Minimal APIs](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis)
- [Entity Framework Core - Querying Data](https://docs.microsoft.com/en-us/ef/core/querying/)
- [Entity Framework Core - Loading Related Data](https://docs.microsoft.com/en-us/ef/core/querying/related-data/)
- [LINQ (Language Integrated Query)](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/linq/)