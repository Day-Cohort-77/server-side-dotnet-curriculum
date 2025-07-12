# Implementing Endpoints with DTOs

In this chapter, we'll implement the endpoint methods for our Loncotes Library API. We'll build on the endpoint classes we created in the previous chapter and add the actual implementation code.

## Implementing Material Endpoints

Now, let's implement the endpoint methods for the `MaterialEndpoints` class:

```csharp
// Endpoints/MaterialEndpoints.cs
// Add these methods inside the MaterialEndpoints class

private static IResult GetAllMaterials(LoncotesLibraryDbContext db, IMapper mapper)
{
    var materials = db.Materials
        .Include(m => m.Genre)
        .Include(m => m.MaterialType)
        .Where(m => m.OutOfCirculationSince == null)
        .ToList();

    return Results.Ok(mapper.Map<List<MaterialDto>>(materials));
}

private static IResult GetMaterialById(LoncotesLibraryDbContext db, IMapper mapper, int id)
{
    var material = db.Materials
        .Include(m => m.Genre)
        .Include(m => m.MaterialType)
        .Include(m => m.Checkouts)
            .ThenInclude(c => c.Patron)
        .FirstOrDefault(m => m.Id == id);

    if (material == null)
    {
        return Results.NotFound();
    }

    return Results.Ok(mapper.Map<MaterialDto>(material));
}

private static IResult CreateMaterial(LoncotesLibraryDbContext db, IMapper mapper, MaterialDto materialDto)
{
    var material = new Material
    {
        MaterialName = materialDto.MaterialName,
        MaterialTypeId = materialDto.MaterialTypeId,
        GenreId = materialDto.GenreId
    };

    db.Materials.Add(material);
    db.SaveChanges();

    return Results.Created($"/materials/{material.Id}", mapper.Map<MaterialDto>(material));
}

private static IResult RemoveMaterialFromCirculation(LoncotesLibraryDbContext db, int id)
{
    var material = db.Materials.Find(id);

    if (material == null)
    {
        return Results.NotFound();
    }

    material.OutOfCirculationSince = DateTime.Now;
    db.SaveChanges();

    return Results.NoContent();
}
```

## Implementing Patron Endpoints

Now, let's implement the endpoint methods for the `PatronEndpoints` class:

```csharp
// Endpoints/PatronEndpoints.cs
// Add these methods inside the PatronEndpoints class

private static IResult GetAllPatrons(LoncotesLibraryDbContext db, IMapper mapper)
{
    var patrons = db.Patrons.ToList();
    return Results.Ok(mapper.Map<List<PatronDto>>(patrons));
}

private static IResult GetPatronById(LoncotesLibraryDbContext db, IMapper mapper, int id)
{
    var patron = db.Patrons.FirstOrDefault(p => p.Id == id);

    if (patron == null)
    {
        return Results.NotFound();
    }

    return Results.Ok(mapper.Map<PatronDto>(patron));
}

private static IResult UpdatePatron(LoncotesLibraryDbContext db, IMapper mapper, int id, PatronDto patronDto)
{
    var patron = db.Patrons.Find(id);

    if (patron == null)
    {
        return Results.NotFound();
    }

    patron.Address = patronDto.Address;
    patron.Email = patronDto.Email;

    db.SaveChanges();

    return Results.NoContent();
}

private static IResult DeactivatePatron(LoncotesLibraryDbContext db, int id)
{
    var patron = db.Patrons.Find(id);

    if (patron == null)
    {
        return Results.NotFound();
    }

    patron.IsActive = false;
    db.SaveChanges();

    return Results.NoContent();
}
```

## Implementing Checkout Endpoints

Finally, let's implement the endpoint methods for the `CheckoutEndpoints` class:

```csharp
// Endpoints/CheckoutEndpoints.cs
// Add these methods inside the CheckoutEndpoints class

private static IResult GetAllCheckouts(LoncotesLibraryDbContext db, IMapper mapper)
{
    var checkouts = db.Checkouts
        .Include(c => c.Material)
        .Include(c => c.Patron)
        .ToList();

    return Results.Ok(mapper.Map<List<CheckoutDto>>(checkouts));
}

private static IResult CreateCheckout(LoncotesLibraryDbContext db, IMapper mapper, CheckoutDto checkoutDto)
{
    var checkout = new Checkout
    {
        MaterialId = checkoutDto.MaterialId,
        PatronId = checkoutDto.PatronId,
        CheckoutDate = DateTime.Today
    };

    db.Checkouts.Add(checkout);
    db.SaveChanges();

    return Results.Created($"/checkouts/{checkout.Id}", mapper.Map<CheckoutDto>(checkout));
}

private static IResult ReturnCheckout(LoncotesLibraryDbContext db, int id)
{
    var checkout = db.Checkouts.Find(id);

    if (checkout == null)
    {
        return Results.NotFound();
    }

    checkout.ReturnDate = DateTime.Today;
    db.SaveChanges();

    return Results.NoContent();
}
```

## Testing the Endpoints

Now that we've implemented our endpoints, let's test them to make sure they're working correctly.

### Get All Materials

Send a GET request to `/materials` to get all materials that are in circulation. You should see a response like this:

```json
[
  {
    "id": 1,
    "materialName": "The Great Gatsby",
    "materialTypeId": 1,
    "materialType": {
      "id": 1,
      "name": "Book",
      "checkoutDays": 14
    },
    "genreId": 1,
    "genre": {
      "id": 1,
      "name": "Fiction"
    },
    "outOfCirculationSince": null
  },
  // More materials...
]
```

### Get Material by ID

Send a GET request to `/materials/1` to get the details of a specific material. You should see a response like this:

```json
{
  "id": 1,
  "materialName": "The Great Gatsby",
  "materialTypeId": 1,
  "materialType": {
    "id": 1,
    "name": "Book",
    "checkoutDays": 14
  },
  "genreId": 1,
  "genre": {
    "id": 1,
    "name": "Fiction"
  },
  "outOfCirculationSince": null,
  "checkouts": []
}
```

### Get All Patrons

Send a GET request to `/patrons` to get all patrons. You should see a response like this:

```json
[
  {
    "id": 1,
    "firstName": "John",
    "lastName": "Doe",
    "address": "123 Main St",
    "email": "john@example.com",
    "isActive": true,
    "fullName": "John Doe"
  },
  {
    "id": 2,
    "firstName": "Jane",
    "lastName": "Smith",
    "address": "456 Elm St",
    "email": "jane@example.com",
    "isActive": true,
    "fullName": "Jane Smith"
  }
]
```

Notice how the `fullName` property is automatically calculated from the `firstName` and `lastName` properties.

## The Benefits of Using DTOs with AutoMapper

In this chapter, we've seen how DTOs give us flexibility in how we present data to clients. We've also seen how AutoMapper simplifies the mapping between our models and DTOs.

By using DTOs, we can:
1. Control exactly what data is sent to clients
2. Add calculated properties that don't exist in our database models
3. Hide sensitive data from being exposed

And by using AutoMapper, we can:
1. Reduce the amount of mapping code we need to write
2. Make our code more maintainable
3. Focus on the business logic rather than the mapping logic

In the next chapter, we'll explore how to use DTOs to create more complex calculated properties and handle more advanced scenarios.

Up Next: [Custom DTO: Available Materials](./loncotes-dto-available-materials.md)