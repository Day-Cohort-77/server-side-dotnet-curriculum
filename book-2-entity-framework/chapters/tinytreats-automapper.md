# Tiny Treats: Implementing AutoMapper

## Introduction to AutoMapper

As our Tiny Treats application grows, you may have noticed that we're writing a lot of repetitive code to map between our domain models and DTOs. This is particularly evident in our endpoint implementations where we manually map properties from models to DTOs and vice versa.

AutoMapper is a popular library in the .NET ecosystem that simplifies the mapping between different object types. It uses a convention-based approach to automatically map properties from one object to another, reducing the amount of boilerplate code we need to write.

## The Problem with Manual Mapping

Let's look at an example of manual mapping from our product endpoints:

```csharp
// Manual mapping from Product to ProductDto
var productDtos = products.Select(p => new ProductDto
{
    Id = p.Id,
    Name = p.Name,
    Description = p.Description,
    Price = p.Price,
    IsAvailable = p.IsAvailable,
    ImageUrl = p.ImageUrl
}).ToList();
```

And for creating a new product:

```csharp
// Manual mapping from ProductCreateDto to Product
var product = new Product
{
    Name = productDto.Name,
    Description = productDto.Description,
    Price = productDto.Price,
    IsAvailable = productDto.IsAvailable,
    ImageUrl = productDto.ImageUrl
};
```

This approach has several drawbacks:
1. It's verbose and repetitive
2. It's error-prone (easy to miss a property)
3. It's hard to maintain (if you add a property to the model, you need to update all the mapping code)
4. It clutters your business logic with mapping code

## Adding AutoMapper to the Project

Let's add AutoMapper to our project to simplify these mappings. First, we need to add the necessary NuGet packages:

```bash
dotnet add package AutoMapper.Extensions.Microsoft.DependencyInjection
```

This package includes both AutoMapper and the integration with ASP.NET Core's dependency injection system.

## Creating Mapping Profiles

AutoMapper uses profiles to define mapping configurations. Let's create a new folder called `Mapping` in our project and add a profile for each of our entity types:

```csharp
// Mapping/ProductMappingProfile.cs
using AutoMapper;
using TinyTreats.Models;
using TinyTreats.DTO;

namespace TinyTreats.Mapping
{
    public class ProductMappingProfile : Profile
    {
        public ProductMappingProfile()
        {
            // Map from Product to ProductDto
            CreateMap<Product, ProductDto>();

            // Map from ProductCreateDto to Product
            CreateMap<ProductCreateDto, Product>();

            // Map from ProductUpdateDto to Product
            CreateMap<ProductUpdateDto, Product>();
        }
    }
}
```

```csharp
// Mapping/OrderMappingProfile.cs
using AutoMapper;
using TinyTreats.Models;
using TinyTreats.DTO;

namespace TinyTreats.Mapping
{
    public class OrderMappingProfile : Profile
    {
        public OrderMappingProfile()
        {
            // Map from Order to OrderDto
            CreateMap<Order, OrderDto>()
                .ForMember(dest => dest.CustomerName, opt =>
                    opt.MapFrom(src => $"{src.UserProfile.FirstName} {src.UserProfile.LastName}"))
                .ForMember(dest => dest.Items, opt =>
                    opt.MapFrom(src => src.OrderItems));

            // Map from OrderItem to OrderItemDto
            CreateMap<OrderItem, OrderItemDto>()
                .ForMember(dest => dest.ProductName, opt =>
                    opt.MapFrom(src => src.Product.Name))
                .ForMember(dest => dest.ProductPrice, opt =>
                    opt.MapFrom(src => src.Product.Price))
                .ForMember(dest => dest.Subtotal, opt =>
                    opt.MapFrom(src => src.Quantity * src.Product.Price));

            // Map from OrderCreateDto to Order
            CreateMap<OrderCreateDto, Order>();

            // Map from OrderItemCreateDto to OrderItem
            CreateMap<OrderItemCreateDto, OrderItem>();
        }
    }
}
```

```csharp
// Mapping/UserProfileMappingProfile.cs
using AutoMapper;
using Microsoft.AspNetCore.Identity;
using TinyTreats.Models;
using TinyTreats.DTO;

namespace TinyTreats.Mapping
{
    public class UserProfileMappingProfile : Profile
    {
        public UserProfileMappingProfile()
        {
            // Map from UserProfile to UserProfileDto
            CreateMap<UserProfile, UserProfileDto>()
                .ForMember(dest => dest.Email, opt =>
                    opt.MapFrom(src => src.IdentityUser.Email));

            // Map from RegistrationDto to UserProfile
            CreateMap<RegistrationDto, UserProfile>()
                .ForMember(dest => dest.FirstName, opt =>
                    opt.MapFrom(src => src.FirstName))
                .ForMember(dest => dest.LastName, opt =>
                    opt.MapFrom(src => src.LastName))
                .ForMember(dest => dest.Address, opt =>
                    opt.MapFrom(src => src.Address));
        }
    }
}
```

## Registering AutoMapper with Dependency Injection

Now that we have our mapping profiles, we need to register AutoMapper with the dependency injection container. Open the `Program.cs` file and add the following code after the database configuration:

```csharp
// Add AutoMapper
builder.Services.AddAutoMapper(typeof(Program).Assembly);
```

This will scan the assembly for all classes that inherit from `Profile` and register them with AutoMapper.

## Using AutoMapper in Endpoints

Now that we have AutoMapper set up, we can use it in our endpoints to simplify our mapping code. Let's update our endpoint classes to use AutoMapper.

### Injecting IMapper

First, we need to inject the `IMapper` interface into our endpoint methods:

```csharp
// Example for product endpoints
app.MapGet("/products", async (TinyTreatsDbContext dbContext, IMapper mapper) =>
{
    // Use mapper here
});
```

### Using AutoMapper for Simple Mappings

For simple mappings, we can use the `Map` method:

```csharp
// Map a single entity
var productDto = mapper.Map<ProductDto>(product);

// Map a collection of entities
var productDtos = mapper.Map<List<ProductDto>>(products);
```

### Using AutoMapper for Updates

For updates, we can use the `Map` method with an existing destination object:

```csharp
// Update an existing entity from a DTO
mapper.Map(productUpdateDto, product);
```

### Using ProjectTo for Efficient Queries

For more efficient queries, we can use the `ProjectTo` method to project directly to DTOs in the database query:

```csharp
using AutoMapper.QueryableExtensions;

// Project to DTOs in the query
var productDtos = await dbContext.Products
    .Where(p => p.IsAvailable)
    .ProjectTo<ProductDto>(mapper.ConfigurationProvider)
    .ToListAsync();
```

This can be more efficient than mapping after the query, as it only selects the properties needed for the DTO.

## Benefits of Using AutoMapper

By implementing AutoMapper in our Tiny Treats application, we gain several benefits:

1. **Reduced boilerplate code**: We no longer need to manually map properties between objects.
2. **Improved maintainability**: When we add new properties to our models or DTOs, we don't need to update mapping code in multiple places.
3. **Fewer bugs**: Manual mapping is error-prone, especially when properties are added or renamed.
4. **Cleaner controller code**: Our endpoints can focus on business logic rather than object mapping.
5. **More efficient queries**: Using `ProjectTo` can optimize our database queries.

## Summary

In this chapter, we've implemented AutoMapper in our Tiny Treats application to simplify the mapping between our domain models and DTOs. We've:

1. Added the AutoMapper NuGet package
2. Created mapping profiles for our entity types
3. Registered AutoMapper with the dependency injection container
4. Learned how to use AutoMapper in our endpoints

In the next chapters, we'll use AutoMapper for all our endpoint definitions.


[Next: Creating the TinyTreatsDbContext](./tinytreats-dbcontext.md)