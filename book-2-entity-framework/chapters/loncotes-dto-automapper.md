# Using AutoMapper with DTOs

In this chapter, we'll introduce AutoMapper, a popular library that simplifies the mapping between domain models and DTOs. This will help us reduce the amount of repetitive code in our application.

## The Problem with Manual Mapping

In the previous chapters, we've been manually mapping between our domain models and DTOs. For example:

```csharp
return db.Materials
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
```

This approach has several drawbacks:
1. It's verbose and repetitive
2. It's error-prone (easy to miss a property)
3. It's hard to maintain (if you add a property to the model, you need to update all the mapping code)

## Introducing AutoMapper

AutoMapper is a library that simplifies the mapping between objects. It uses a convention-based approach to map properties from one object to another, based on property names.

### Installing AutoMapper

First, let's install the AutoMapper package:

```bash
dotnet add package AutoMapper
dotnet add package AutoMapper.Extensions.Microsoft.DependencyInjection
```

### Configuring AutoMapper

Next, let's create a profile class that defines the mapping between our domain models and DTOs:

```csharp
using AutoMapper;

public class AutoMapperProfiles : Profile
{
    public AutoMapperProfiles()
    {
        CreateMap<Material, MaterialDto>();
        CreateMap<MaterialDto, Material>();

        CreateMap<MaterialType, MaterialTypeDto>();
        CreateMap<MaterialTypeDto, MaterialType>();

        CreateMap<Genre, GenreDto>();
        CreateMap<GenreDto, Genre>();

        CreateMap<Patron, PatronDto>();
        CreateMap<PatronDto, Patron>();

        CreateMap<Checkout, CheckoutDto>();
        CreateMap<CheckoutDto, Checkout>();

        CreateMap<Checkout, CheckoutWithLateFeeDto>();
        CreateMap<CheckoutWithLateFeeDto, Checkout>();

        CreateMap<Patron, PatronWithBalanceDto>();
        CreateMap<PatronWithBalanceDto, Patron>();

        CreateMap<Material, AvailableMaterialDto>();
        CreateMap<AvailableMaterialDto, Material>();
    }
}
```

This profile defines bidirectional mappings between our domain models and DTOs.

### Registering AutoMapper with Dependency Injection

Now, let's register AutoMapper with the dependency injection container in `Program.cs`:

```csharp
using AutoMapper;

// Add AutoMapper
builder.Services.AddAutoMapper(typeof(AutoMapperProfiles));
```

## Using AutoMapper in Our Endpoints

Now that we have AutoMapper configured, let's update our endpoints to use it.

### Get All Materials

```csharp
app.MapGet("/materials", (LoncotesLibraryDbContext db, IMapper mapper) =>
{
    var materials = db.Materials
        .Include(m => m.Genre)
        .Include(m => m.MaterialType)
        .Where(m => m.OutOfCirculationSince == null)
        .ToList();

    return mapper.Map<List<MaterialDto>>(materials);
});
```

### Get Material Details

```csharp
app.MapGet("/materials/{id}", (LoncotesLibraryDbContext db, IMapper mapper, int id) =>
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
});
```

### Get Available Materials

```csharp
app.MapGet("/materials/available", (LoncotesLibraryDbContext db, IMapper mapper) =>
{
    var materials = db.Materials
        .Include(m => m.Genre)
        .Include(m => m.MaterialType)
        .Include(m => m.Checkouts)
        .Where(m => m.OutOfCirculationSince == null)
        .Where(m => m.Checkouts.All(co => co.ReturnDate != null))
        .ToList();

    return mapper.Map<List<AvailableMaterialDto>>(materials);
});
```

### Get Overdue Checkouts

```csharp
app.MapGet("/checkouts/overdue", (LoncotesLibraryDbContext db, IMapper mapper) =>
{
    var checkouts = db.Checkouts
        .Include(c => c.Material)
            .ThenInclude(m => m.MaterialType)
        .Include(c => c.Patron)
        .Where(c => c.ReturnDate == null)
        .Where(c => (DateTime.Today - c.CheckoutDate).Days > c.Material.MaterialType.CheckoutDays)
        .ToList();

    return mapper.Map<List<OverdueCheckoutDto>>(checkouts);
});
```

### Get Patron with Balance

```csharp
app.MapGet("/patrons/{id}/balance", (LoncotesLibraryDbContext db, IMapper mapper, int id) =>
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

    return Results.Ok(mapper.Map<PatronWithBalanceDto>(patron));
});
```

## Using AutoMapper's ProjectTo for More Efficient Queries

AutoMapper provides a `ProjectTo` method that can be used with LINQ queries to project directly to DTOs. This can be more efficient than mapping after the query, as it only selects the properties needed for the DTO.

First, add the necessary using statement:

```csharp
using AutoMapper.QueryableExtensions;
```

Then, update our endpoints to use `ProjectTo`:

### Get All Materials

```csharp
app.MapGet("/materials", (LoncotesLibraryDbContext db, IMapper mapper) =>
{
    return db.Materials
        .Where(m => m.OutOfCirculationSince == null)
        .ProjectTo<MaterialDto>(mapper.ConfigurationProvider)
        .ToList();
});
```

### Get Material Types

```csharp
app.MapGet("/materialtypes", (LoncotesLibraryDbContext db, IMapper mapper) =>
{
    return db.MaterialTypes
        .ProjectTo<MaterialTypeDto>(mapper.ConfigurationProvider)
        .ToList();
});
```

### Get Genres

```csharp
app.MapGet("/genres", (LoncotesLibraryDbContext db, IMapper mapper) =>
{
    return db.Genres
        .ProjectTo<GenreDto>(mapper.ConfigurationProvider)
        .ToList();
});
```

### Get Patrons

```csharp
app.MapGet("/patrons", (LoncotesLibraryDbContext db, IMapper mapper) =>
{
    return db.Patrons
        .ProjectTo<PatronDto>(mapper.ConfigurationProvider)
        .ToList();
});
```

## Benefits of Using AutoMapper

Using AutoMapper provides several benefits:

1. **Reduced Code**: Less mapping code means less to write and maintain
2. **Consistency**: Mapping is handled consistently throughout the application
3. **Maintainability**: When you add a property to a model or DTO, you don't need to update all the mapping code
4. **Performance**: `ProjectTo` can optimize queries to only select the properties needed for the DTO

## Conclusion

In this chapter, we've seen how AutoMapper can simplify the mapping between our domain models and DTOs. By using AutoMapper, we've reduced the amount of repetitive code in our application and made it more maintainable.

DTOs are a powerful pattern for shaping the data that flows between our API and clients. They allow us to:

1. Control exactly what data is exposed to clients
2. Add calculated properties that don't exist in our database models
3. Customize our responses for specific use cases
4. Encapsulate complex business logic

Combined with AutoMapper, DTOs provide a flexible and maintainable way to handle data transfer in our application.

## Next Steps

Now that you've completed this series of chapters on DTOs, you should have a good understanding of how to use them effectively in your ASP.NET Core applications. You can apply these patterns to other projects, such as the Creek River Campground API or Hillary's Hair Care.

Remember that DTOs are just one tool in your toolbox. Use them when they provide value, but don't feel obligated to use them for every endpoint. Sometimes, returning the domain model directly is the simplest and most appropriate solution.

Happy coding!