# Quieter Code with AutoMapper

In this chapter, we'll explore how to use AutoMapper more effectively to simplify our code even further. We've already been using AutoMapper in our project, but there are additional features and techniques that can make our code even cleaner and more maintainable.

## The Problem with Manual Mapping

Throughout this project, we've been using AutoMapper to map between our domain models and DTOs. This has already saved us a lot of code compared to manual mapping, which would look something like this:

```csharp
// Manual mapping example
var materialDtos = materials.Select(m => new MaterialDto
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
}).ToList();
```

This approach has several drawbacks:
1. It's verbose and repetitive
2. It's error-prone (easy to miss a property)
3. It's hard to maintain (if you add a property to the model, you need to update all the mapping code)

## Using AutoMapper's ProjectTo for More Efficient Queries

So far, we've been using AutoMapper's `Map` method to map objects after we've retrieved them from the database. This works well, but it means we're retrieving all properties from the database, even if we don't need them all.

AutoMapper provides a `ProjectTo` method that can be used with LINQ queries to project directly to DTOs. This can be more efficient than mapping after the query, as it only selects the properties needed for the DTO.

First, add the necessary using statement:

```csharp
using AutoMapper.QueryableExtensions;
```

Then, update our endpoints to use `ProjectTo`:

### Get All Materials

```csharp
// Endpoints/MaterialEndpoints.cs
private static IResult GetAllMaterials(LoncotesLibraryDbContext db, IMapper mapper)
{
    return Results.Ok(db.Materials
        .Include(m => m.Genre)
        .Include(m => m.MaterialType)
        .Where(m => m.OutOfCirculationSince == null)
        .ProjectTo<MaterialDto>(mapper.ConfigurationProvider)
        .ToList());
}
```

### Get Material Types

```csharp
// Add this to the MapMaterialEndpoints method
app.MapGet("/materialtypes", GetMaterialTypes);

// Add this method to the MaterialEndpoints class
private static IResult GetMaterialTypes(LoncotesLibraryDbContext db, IMapper mapper)
{
    return Results.Ok(db.MaterialTypes
        .ProjectTo<MaterialTypeDto>(mapper.ConfigurationProvider)
        .ToList());
}
```

### Get Genres

```csharp
// Add this to the MapMaterialEndpoints method
app.MapGet("/genres", GetGenres);

// Add this method to the MaterialEndpoints class
private static IResult GetGenres(LoncotesLibraryDbContext db, IMapper mapper)
{
    return Results.Ok(db.Genres
        .ProjectTo<GenreDto>(mapper.ConfigurationProvider)
        .ToList());
}
```

### Get Patrons

```csharp
// Endpoints/PatronEndpoints.cs
private static IResult GetAllPatrons(LoncotesLibraryDbContext db, IMapper mapper)
{
    return Results.Ok(db.Patrons
        .ProjectTo<PatronDto>(mapper.ConfigurationProvider)
        .ToList());
}
```

## Handling Circular References

When mapping objects with circular references (e.g., a Material has Checkouts, and each Checkout has a Material), AutoMapper can get stuck in an infinite loop. To handle this, we can use the `ExplicitExpansion()` option:

```csharp
// Mapping/AutoMapperProfiles.cs
public class AutoMapperProfiles : Profile
{
    public AutoMapperProfiles()
    {
        CreateMap<Material, MaterialDto>()
            .ForMember(dest => dest.Checkouts, opt => opt.ExplicitExpansion());

        CreateMap<MaterialType, MaterialTypeDto>();
        CreateMap<Genre, GenreDto>();

        CreateMap<Patron, PatronDto>()
            .ForMember(dest => dest.Checkouts, opt => opt.ExplicitExpansion());

        CreateMap<Checkout, CheckoutDto>()
            .ForMember(dest => dest.Material, opt => opt.ExplicitExpansion())
            .ForMember(dest => dest.Patron, opt => opt.ExplicitExpansion());

        CreateMap<Material, AvailableMaterialDto>();
        CreateMap<Checkout, OverdueCheckoutDto>();
        CreateMap<Checkout, CheckoutWithLateFeeDto>();
        CreateMap<Patron, PatronWithBalanceDto>();
    }
}
```

The `ExplicitExpansion` option tells AutoMapper to only include the specified property when explicitly requested. This prevents infinite loops when mapping circular references.

## Using AutoMapper Profiles for Different Use Cases

As our application grows, we might want to have different mappings for different use cases. For example, we might want a simplified version of a Material for a list view, and a detailed version for a detail view.

Create simplified DTOs for list views:

```csharp
// DTOs/MaterialListDto.cs
public class MaterialListDto
{
    public int Id { get; set; }
    public string MaterialName { get; set; }
    public string MaterialTypeName { get; set; }
    public string GenreName { get; set; }
}

// DTOs/PatronListDto.cs
public class PatronListDto
{
    public int Id { get; set; }
    public string FullName { get; set; }
    public bool IsActive { get; set; }
}
```


Then create multiple profiles for different use cases:

```csharp
// Mapping/AutoMapperProfiles.cs
public class AutoMapperProfiles : Profile
{
    public AutoMapperProfiles()
    {
        // Basic mappings
        CreateMap<Material, MaterialDto>();
        CreateMap<MaterialType, MaterialTypeDto>();
        CreateMap<Genre, GenreDto>();
        CreateMap<Patron, PatronDto>();
        CreateMap<Checkout, CheckoutDto>();

        // Specialized mappings
        CreateMap<Material, AvailableMaterialDto>();
        CreateMap<Checkout, OverdueCheckoutDto>();
        CreateMap<Checkout, CheckoutWithLateFeeDto>();
        CreateMap<Patron, PatronWithBalanceDto>();

        // Simplified mappings for list views
        CreateMap<Material, MaterialListDto>()
            .ForMember(dest => dest.MaterialTypeName, opt => opt.MapFrom(src => src.MaterialType.Name))
            .ForMember(dest => dest.GenreName, opt => opt.MapFrom(src => src.Genre.Name));

        CreateMap<Patron, PatronListDto>()
            .ForMember(dest => dest.FullName, opt => opt.MapFrom(src => $"{src.FirstName} {src.LastName}"));
    }
}
```

And use them in our endpoints:

```csharp
// Endpoints/MaterialEndpoints.cs
// Add this to the MapMaterialEndpoints method
app.MapGet("/materials/list", GetMaterialsList);

// Add this method to the MaterialEndpoints class
private static IResult GetMaterialsList(LoncotesLibraryDbContext db, IMapper mapper)
{
    return Results.Ok(db.Materials
        .Include(m => m.Genre)
        .Include(m => m.MaterialType)
        .Where(m => m.OutOfCirculationSince == null)
        .ProjectTo<MaterialListDto>(mapper.ConfigurationProvider)
        .ToList());
}
```

## Benefits of Using AutoMapper

Using AutoMapper provides several benefits:

1. **Reduced Code**: Less mapping code means less to write and maintain
2. **Consistency**: Mapping is handled consistently throughout the application
3. **Maintainability**: When you add a property to a model or DTO, you don't need to update all the mapping code
4. **Performance**: `ProjectTo` can optimize queries to only select the properties needed for the DTO

## Conclusion

In this chapter, we've explored how to use AutoMapper more effectively to simplify our code even further. We've seen how to use `ProjectTo` for more efficient queries, how to handle circular references, and how to create multiple profiles for different use cases.

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