# Basic Features with DTOs

In this chapter, we'll implement the basic features for the Loncotes Library API using DTOs to shape our responses. We'll see how DTOs give us flexibility in how we present data to clients.

## Get All Materials

Let's start by implementing an endpoint to get all materials that are in circulation. We'll use DTOs to shape our response:

```csharp
app.MapGet("/materials", (LoncotesLibraryDbContext db) =>
{
    return db.Materials
        .Include(m => m.Genre)
        .Include(m => m.MaterialType)
        .Where(m => m.OutOfCirculationSince == null)
        .Select(m => new MaterialDto
        {
            Id = m.Id,
            MaterialName = m.MaterialName,
            MaterialTypeId = m.MaterialTypeId,
            MaterialType = new MaterialTypeDto
            {
                Id = m.MaterialType.Id,
                Name = m.MaterialType.Name,
                CheckoutDays = m.MaterialType.CheckoutDays
            },
            GenreId = m.GenreId,
            Genre = new GenreDto
            {
                Id = m.Genre.Id,
                Name = m.Genre.Name
            },
            OutOfCirculationSince = m.OutOfCirculationSince
        })
        .ToList();
});
```

Notice how we're using the `Select` method to transform each `Material` entity into a `MaterialDto`. This gives us complete control over what data is included in the response.

## Get Materials by Genre and/or MaterialType

Now, let's enhance our endpoint to filter materials by genre and/or material type:

```csharp
app.MapGet("/materials", (LoncotesLibraryDbContext db, int? genreId, int? materialTypeId) =>
{
    var materials = db.Materials
        .Include(m => m.Genre)
        .Include(m => m.MaterialType)
        .Where(m => m.OutOfCirculationSince == null);

    // Apply filters if provided
    if (genreId != null)
    {
        materials = materials.Where(m => m.GenreId == genreId);
    }

    if (materialTypeId != null)
    {
        materials = materials.Where(m => m.MaterialTypeId == materialTypeId);
    }

    return materials
        .Select(m => new MaterialDto
        {
            Id = m.Id,
            MaterialName = m.MaterialName,
            MaterialTypeId = m.MaterialTypeId,
            MaterialType = new MaterialTypeDto
            {
                Id = m.MaterialType.Id,
                Name = m.MaterialType.Name,
                CheckoutDays = m.MaterialType.CheckoutDays
            },
            GenreId = m.GenreId,
            Genre = new GenreDto
            {
                Id = m.Genre.Id,
                Name = m.Genre.Name
            },
            OutOfCirculationSince = m.OutOfCirculationSince
        })
        .ToList();
});
```

This endpoint now accepts optional query parameters for `genreId` and `materialTypeId`, and filters the results accordingly.

## Get Material Details

Let's implement an endpoint to get details for a specific material, including its checkouts:

```csharp
app.MapGet("/materials/{id}", (LoncotesLibraryDbContext db, int id) =>
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

    return Results.Ok(new MaterialDto
    {
        Id = material.Id,
        MaterialName = material.MaterialName,
        MaterialTypeId = material.MaterialTypeId,
        MaterialType = new MaterialTypeDto
        {
            Id = material.MaterialType.Id,
            Name = material.MaterialType.Name,
            CheckoutDays = material.MaterialType.CheckoutDays
        },
        GenreId = material.GenreId,
        Genre = new GenreDto
        {
            Id = material.Genre.Id,
            Name = material.Genre.Name
        },
        OutOfCirculationSince = material.OutOfCirculationSince,
        Checkouts = material.Checkouts.Select(c => new CheckoutDto
        {
            Id = c.Id,
            MaterialId = c.MaterialId,
            PatronId = c.PatronId,
            Patron = new PatronDto
            {
                Id = c.Patron.Id,
                FirstName = c.Patron.FirstName,
                LastName = c.Patron.LastName,
                Address = c.Patron.Address,
                Email = c.Patron.Email,
                IsActive = c.Patron.IsActive
                // FullName is automatically calculated
            },
            CheckoutDate = c.CheckoutDate,
            ReturnDate = c.ReturnDate
        }).ToList()
    });
});
```

Notice how we're including the `Patron` for each checkout, and how the `FullName` property in `PatronDto` is automatically calculated.

## Add a Material

Now, let's implement an endpoint to add a new material:

```csharp
app.MapPost("/materials", (LoncotesLibraryDbContext db, MaterialDto materialDto) =>
{
    var material = new Material
    {
        MaterialName = materialDto.MaterialName,
        MaterialTypeId = materialDto.MaterialTypeId,
        GenreId = materialDto.GenreId
    };

    db.Materials.Add(material);
    db.SaveChanges();

    // Return the newly created material with its ID
    return Results.Created($"/materials/{material.Id}", new MaterialDto
    {
        Id = material.Id,
        MaterialName = material.MaterialName,
        MaterialTypeId = material.MaterialTypeId,
        GenreId = material.GenreId,
        OutOfCirculationSince = material.OutOfCirculationSince
    });
});
```

Here, we're accepting a `MaterialDto` from the client, using it to create a new `Material` entity, and then returning a new `MaterialDto` with the generated ID.

## Remove a Material From Circulation

Let's implement an endpoint to remove a material from circulation:

```csharp
app.MapPut("/materials/{id}/remove-from-circulation", (LoncotesLibraryDbContext db, int id) =>
{
    var material = db.Materials.Find(id);

    if (material == null)
    {
        return Results.NotFound();
    }

    material.OutOfCirculationSince = DateTime.Now;
    db.SaveChanges();

    return Results.NoContent();
});
```

This endpoint performs a "soft delete" by setting the `OutOfCirculationSince` property to the current date and time.

## Get Material Types

Let's implement an endpoint to get all material types:

```csharp
app.MapGet("/materialtypes", (LoncotesLibraryDbContext db) =>
{
    return db.MaterialTypes
        .Select(mt => new MaterialTypeDto
        {
            Id = mt.Id,
            Name = mt.Name,
            CheckoutDays = mt.CheckoutDays
        })
        .ToList();
});
```

## Get Genres

Similarly, let's implement an endpoint to get all genres:

```csharp
app.MapGet("/genres", (LoncotesLibraryDbContext db) =>
{
    return db.Genres
        .Select(g => new GenreDto
        {
            Id = g.Id,
            Name = g.Name
        })
        .ToList();
});
```

## Get Patrons

Let's implement an endpoint to get all patrons:

```csharp
app.MapGet("/patrons", (LoncotesLibraryDbContext db) =>
{
    return db.Patrons
        .Select(p => new PatronDto
        {
            Id = p.Id,
            FirstName = p.FirstName,
            LastName = p.LastName,
            Address = p.Address,
            Email = p.Email,
            IsActive = p.IsActive
            // FullName is automatically calculated
        })
        .ToList();
});
```

Notice how the `FullName` property is automatically calculated for each patron.

## Get Patron with Checkouts

Let's implement an endpoint to get a patron with their checkouts:

```csharp
app.MapGet("/patrons/{id}", (LoncotesLibraryDbContext db, int id) =>
{
    var patron = db.Patrons
        .Include(p => p.Checkouts)
            .ThenInclude(c => c.Material)
                .ThenInclude(m => m.MaterialType)
        .FirstOrDefault(p => p.Id == id);

    if (patron == null)
    {
        return Results.NotFound();
    }

    return Results.Ok(new PatronDto
    {
        Id = patron.Id,
        FirstName = patron.FirstName,
        LastName = patron.LastName,
        Address = patron.Address,
        Email = patron.Email,
        IsActive = patron.IsActive,
        // FullName is automatically calculated
        Checkouts = patron.Checkouts.Select(c => new CheckoutDto
        {
            Id = c.Id,
            MaterialId = c.MaterialId,
            Material = new MaterialDto
            {
                Id = c.Material.Id,
                MaterialName = c.Material.MaterialName,
                MaterialTypeId = c.Material.MaterialTypeId,
                MaterialType = new MaterialTypeDto
                {
                    Id = c.Material.MaterialType.Id,
                    Name = c.Material.MaterialType.Name,
                    CheckoutDays = c.Material.MaterialType.CheckoutDays
                },
                GenreId = c.Material.GenreId,
                OutOfCirculationSince = c.Material.OutOfCirculationSince
            },
            PatronId = c.PatronId,
            CheckoutDate = c.CheckoutDate,
            ReturnDate = c.ReturnDate
        }).ToList()
    });
});
```

## Update Patron

Let's implement an endpoint to update a patron's address and email:

```csharp
app.MapPut("/patrons/{id}", (LoncotesLibraryDbContext db, int id, PatronDto patronDto) =>
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
});
```

## Deactivate Patron

Let's implement an endpoint to deactivate a patron:

```csharp
app.MapPut("/patrons/{id}/deactivate", (LoncotesLibraryDbContext db, int id) =>
{
    var patron = db.Patrons.Find(id);

    if (patron == null)
    {
        return Results.NotFound();
    }

    patron.IsActive = false;
    db.SaveChanges();

    return Results.NoContent();
});
```

## Checkout a Material

Let's implement an endpoint to checkout a material:

```csharp
app.MapPost("/checkouts", (LoncotesLibraryDbContext db, CheckoutDto checkoutDto) =>
{
    var checkout = new Checkout
    {
        MaterialId = checkoutDto.MaterialId,
        PatronId = checkoutDto.PatronId,
        CheckoutDate = DateTime.Today
    };

    db.Checkouts.Add(checkout);
    db.SaveChanges();

    return Results.Created($"/checkouts/{checkout.Id}", new CheckoutDto
    {
        Id = checkout.Id,
        MaterialId = checkout.MaterialId,
        PatronId = checkout.PatronId,
        CheckoutDate = checkout.CheckoutDate,
        ReturnDate = checkout.ReturnDate
    });
});
```

## Return a Material

Finally, let's implement an endpoint to return a material:

```csharp
app.MapPut("/checkouts/{id}/return", (LoncotesLibraryDbContext db, int id) =>
{
    var checkout = db.Checkouts.Find(id);

    if (checkout == null)
    {
        return Results.NotFound();
    }

    checkout.ReturnDate = DateTime.Today;
    db.SaveChanges();

    return Results.NoContent();
});
```

In this chapter, we've implemented all the basic features for the Loncotes Library API using DTOs to shape our responses. In the next chapter, we'll explore how to use DTOs to create more complex calculated properties.

Up Next: [Available Materials with DTOs](./loncotes-dto-available-materials.md)