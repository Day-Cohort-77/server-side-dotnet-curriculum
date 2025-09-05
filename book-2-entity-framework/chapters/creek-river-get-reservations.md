# Chapter to GET reservations which introduces LINQ methods ThenInclude

In this chapter, we'll create an API endpoint to retrieve reservations from our database using Entity Framework Core. We'll learn how to include related data from multiple levels of relationships using the `Include` and `ThenInclude` methods.

## Understanding Related Data in Entity Framework Core

One of the powerful features of Entity Framework Core is its ability to load related data along with the primary entities you're querying. This is similar to performing JOINs in SQL, but with a more object-oriented approach.

EF Core provides several methods for loading related data:

1. **Eager Loading**: Loading related data along with the primary entities in the same query using the `Include` and `ThenInclude` methods.
2. **Explicit Loading**: Loading related data explicitly after the primary entities have been loaded.
3. **Lazy Loading**: Loading related data automatically when the navigation property is accessed.

In this chapter, we'll focus on eager loading using `Include` and `ThenInclude`.

## Understanding Include and ThenInclude

The `Include` method is used to specify related entities to include in the query results. For example, when querying reservations, you might want to include the related campsite and user profile.

The `ThenInclude` method is used to specify additional related entities to include from the included entities. For example, after including the campsite, you might want to include the campsite type.

## Creating the GET Endpoint for Reservations

Let's create an endpoint to retrieve all reservations with their related data:

1. Open the `ReservationEndpoints.cs` file.

2. Add the following endpoint after the existing endpoints:

```csharp
app.MapGet("/reservations", (CreekRiverDbContext db) =>
{
    return db.Reservations
        .Include(r => r.UserProfile)
        .Include(r => r.Campsite)
        .ThenInclude(c => c.CampsiteType)
        .OrderBy(res => res.CheckinDate)
        .ToList();
});
```

Let's break down this code:

- `app.MapGet("/reservations", ...)`: This maps HTTP GET requests to the `/reservations` URL to our handler function.

- `(CreekRiverDbContext db) => { ... }`: This is the handler function. The `CreekRiverDbContext` parameter is injected by ASP.NET Core's dependency injection system.

- `db.Reservations`: This accesses the `Reservations` DbSet in our DbContext, which represents the `Reservations` table in the database.

- `.Include(r => r.UserProfile)`: This tells EF Core to include the related `UserProfile` entity for each reservation. This is similar to a LEFT JOIN in SQL.

- `.Include(r => r.Campsite)`: This tells EF Core to include the related `Campsite` entity for each reservation.

- `.ThenInclude(c => c.CampsiteType)`: This tells EF Core to include the related `CampsiteType` entity for each campsite. This is where we start to see the power of `ThenInclude`: it allows us to include entities that are related to the entities we've already included.

- `.OrderBy(res => res.CheckinDate)`: This orders the results by the check-in date.

- `.ToList()`: This executes the query and returns all reservations as a list, including their related data.

## Requesting Reservations with Yaak

Perform a GET request to the `/reservations` endpoint. You probably don't see what you expect. In all likelihood, you'll get an exception in the response body that starts with **System.Text.Json.JsonException: A possible object cycle was detected.**

This is because in the LINQ statement, you join the **Campsites** table. The problem is that the **Campsite** model has a navigation property for reservations.

```cs
public List<Reservation> Reservations { get; set; }
```

The the query that LINQ is building for you joins the campsites table, which requires that the reservation table be joined. The **Reservation** model also has a navigation property to the campsite.

```cs
public Campsite Campsite { get; set; }
```

This means that when you join the reservation table, the campsite table must be joined, which joins the reservation table, which joins the campsite table, which joins the reservations table... well, I think you get the point now.

To avoid this infinite joining, you can add a new configuration to the `Program.cs` module—right before the `var app = builder.Build();` statement.

```cs
builder.Services.Configure<Microsoft.AspNetCore.Http.Json.JsonOptions>(options =>
{
    options.SerializerOptions.ReferenceHandler = System.Text.Json.Serialization.ReferenceHandler.IgnoreCycles;
});
```

Then restart your API and get reservations with Yaak again. Then you should see the reservations that were seeded for campsite 1.

## Understanding the Generated SQL Query

With that configuration added, the LINQ query in our endpoint is translated into a SQL query by Entity Framework Core. The generated SQL query might look something like this:

```sql
SELECT r.Id, r.CampsiteId, r.UserProfileId, r.CheckinDate, r.CheckoutDate,
       up.Id, up.FirstName, up.LastName, up.Email,
       c.Id, c.Nickname, c.ImageUrl, c.CampsiteTypeId,
       ct.Id, ct.CampsiteTypeName, ct.MaxReservationDays, ct.FeePerNight
FROM Reservations r
LEFT JOIN UserProfiles up ON r.UserProfileId = up.Id
LEFT JOIN Campsites c ON r.CampsiteId = c.Id
LEFT JOIN CampsiteTypes ct ON c.CampsiteTypeId = ct.Id
ORDER BY r.CheckinDate
```

This SQL query joins the `Reservations` table with the `UserProfiles`, `Campsites`, and `CampsiteTypes` tables, and selects all the columns from these tables.

## Understanding the Response

The response from our endpoint will be a JSON array of reservation objects, each with its related user profile, campsite, and campsite type:

```json
[
  {
    "id": 1,
    "campsiteId": 1,
    "userProfileId": 1,
    "checkinDate": "2023-06-10T00:00:00",
    "checkoutDate": "2023-06-13T00:00:00",
    "userProfile": {
      "id": 1,
      "firstName": "Admina",
      "lastName": "Strator",
      "email": "admina@creekriver.campground",
      "reservations": []
    },
    "campsite": {
      "id": 1,
      "nickname": "Barred Owl",
      "imageUrl": "https://tnstateparks.com/assets/images/content-images/campgrounds/249/colsp-area2-site73.jpg",
      "campsiteTypeId": 1,
      "campsiteType": {
        "id": 1,
        "campsiteTypeName": "Tent",
        "maxReservationDays": 7,
        "feePerNight": 15.99,
        "campsites": []
      },
      "reservations": []
    }
  },
  // More reservations...
]
```

## Conclusion

In this chapter, we've created an API endpoint to retrieve reservations with their related data using Entity Framework Core. We've learned how to:

1. Use the `Include` method to include related entities in the query results
2. Use the `ThenInclude` method to include additional related entities from the included entities
3. Configure JSON serialization to handle circular references
4. Handle circular references in the entity model
5. Add filtering and pagination to the endpoint

These concepts are fundamental to creating efficient and flexible APIs with ASP.NET Core and Entity Framework Core.

In the next chapter, we'll create an endpoint to create a new reservation.

Up Next: [Chapter to create a reservation](./creek-river-create-reservation.md)

## 🔍 Additional Materials

- [Entity Framework Core - Loading Related Data](https://docs.microsoft.com/en-us/ef/core/querying/related-data/)
- [Entity Framework Core - Complex Query Operators](https://docs.microsoft.com/en-us/ef/core/querying/complex-query-operators)
- [ASP.NET Core - JSON serialization](https://docs.microsoft.com/en-us/aspnet/core/web-api/advanced/formatting)
- [LINQ - Query Syntax](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/linq/query-syntax-and-method-syntax-in-linq)