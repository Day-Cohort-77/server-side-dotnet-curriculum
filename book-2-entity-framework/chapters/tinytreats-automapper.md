# Tiny Treats: Implementing AutoMapper

As your Tiny Treats application grows, you would end up writing a lot of repetitive code to map between our domain models and DTOs in your endpoint methods. To eliminate this from the start, you will install and use AutoMapper—which significantly reduces the noise of your code.

## Adding AutoMapper to the Project

Add AutoMapper to our project to simplify these mappings. First, you need to add the necessary NuGet packages:

```bash
dotnet add package AutoMapper.Extensions.Microsoft.DependencyInjection
```

This package includes both AutoMapper and the integration with ASP.NET Core's dependency injection system.

## Creating Mapping Profiles

AutoMapper uses profiles to define mapping configurations. Create a new folder called `Mapping` in your project and add a profile for each of your entity types:

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

## Summary

In this chapter, we've implemented AutoMapper in our Tiny Treats application to simplify the mapping between our domain models and DTOs. We've:

1. Added the AutoMapper NuGet package
2. Created mapping profiles for our entity types
3. Registered AutoMapper with the dependency injection container
4. Learned how to use AutoMapper in our endpoints

In the next chapters, we'll use AutoMapper for all our endpoint definitions.


[Next: Creating the TinyTreatsDbContext](./tinytreats-dbcontext.md)