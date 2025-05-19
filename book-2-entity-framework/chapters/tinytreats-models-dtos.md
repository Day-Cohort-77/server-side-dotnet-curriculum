# TinyTreats Models and DTOs

In this chapter, we'll define all the database models and Data Transfer Objects (DTOs) needed for our TinyTreats bakery management system. Having a clear understanding of our data model is essential before we implement authentication, authorization, and API endpoints.

## Understanding Models and DTOs

Before we dive into the implementation, let's clarify the difference between models and DTOs:

- **Models** represent the database tables and relationships in our application. They are used by Entity Framework Core to create and interact with the database.
- **DTOs (Data Transfer Objects)** are simplified objects used to transfer data between the client and server. They often contain a subset of model properties and help decouple our API from our database schema.

## Creating the User-Related Models

First, let's create our `UserProfile` model that extends the Identity user information:

```csharp
// Models/UserProfile.cs
using Microsoft.AspNetCore.Identity;

namespace TinyTreats.Models;

public class UserProfile
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Address { get; set; }

    // This connects to the ASP.NET Core Identity user
    public string IdentityUserId { get; set; }
    public IdentityUser IdentityUser { get; set; }

    // Navigation property for orders
    public List<Order> Orders { get; set; }
}
```

The `UserProfile` model extends the basic information stored in the ASP.NET Core Identity system. It includes:
- Basic personal information (name, address)
- A reference to the Identity user (for authentication)
- A navigation property for orders (to easily access a user's orders)

## Creating the Product Model

Next, let's create the `Product` model for our bakery items:

```csharp
// Models/Product.cs
using System.ComponentModel.DataAnnotations;

namespace TinyTreats.Models;

public class Product
{
    public int Id { get; set; }

    [Required]
    [MaxLength(100)]
    public string Name { get; set; }

    [MaxLength(500)]
    public string Description { get; set; }

    [Required]
    [Range(0.01, 1000.00)]
    public decimal Price { get; set; }

    [Required]
    public bool IsAvailable { get; set; } = true;

    [MaxLength(255)]
    public string ImageUrl { get; set; }

    // Navigation property for order items
    public List<OrderItem> OrderItems { get; set; }
}
```

The `Product` model represents bakery items with:
- Basic product information (name, description, price)
- Availability status
- An optional image URL
- Data annotations for validation
- A navigation property for order items

## Creating the Order-Related Models

Now, let's create the `Order` and `OrderItem` models:

```csharp
// Models/Order.cs
using System.ComponentModel.DataAnnotations;

namespace TinyTreats.Models;

public class Order
{
    public int Id { get; set; }

    [Required]
    public DateTime OrderDate { get; set; }

    public DateTime? DeliveryDate { get; set; }

    [Required]
    public string Status { get; set; } // "Pending", "Preparing", "Ready", "Delivered"

    [Required]
    public int UserProfileId { get; set; }
    public UserProfile UserProfile { get; set; }

    public List<OrderItem> OrderItems { get; set; }

    public decimal TotalAmount => OrderItems?.Sum(item => item.Quantity * item.Product.Price) ?? 0;
}

public class OrderItem
{
    public int Id { get; set; }

    [Required]
    public int OrderId { get; set; }
    public Order Order { get; set; }

    [Required]
    public int ProductId { get; set; }
    public Product Product { get; set; }

    [Required]
    [Range(1, int.MaxValue)]
    public int Quantity { get; set; }
}
```

The `Order` model represents a customer's order with:
- Order and delivery dates
- Status tracking
- A reference to the user who placed the order
- A collection of order items
- A calculated total amount

The `OrderItem` model represents individual items within an order with:
- References to both the order and the product
- Quantity of the product ordered

## Creating the DTOs

Now, let's create our DTOs in a separate directory to maintain a clean separation of concerns:

```csharp
// DTOs/AuthDTOs.cs
namespace TinyTreats.DTO;

public class RegistrationDto
{
    public required string Email { get; set; }
    public required string Password { get; set; }
    public required string FirstName { get; set; }
    public required string LastName { get; set; }
    public required string Address { get; set; }
}

public class LoginDto
{
    public required string Email { get; set; }
    public required string Password { get; set; }
}

public class UserProfileDto
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Address { get; set; }
    public string Email { get; set; }
}
```

```csharp
// DTOs/RoleDTOs.cs
namespace TinyTreats.DTO;

// DTO for role creation
public class RoleDto
{
    public string Name { get; set; }
}

// DTO for assigning a role to a user
public class UserRoleDto
{
    public string Email { get; set; }
    public string RoleName { get; set; }
}
```

```csharp
// DTOs/ProductDTOs.cs
namespace TinyTreats.DTO;

public class ProductDto
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
    public decimal Price { get; set; }
    public bool IsAvailable { get; set; }
    public string ImageUrl { get; set; }
}

public class ProductCreateDto
{
    public required string Name { get; set; }
    public string Description { get; set; }
    public required decimal Price { get; set; }
    public bool IsAvailable { get; set; } = true;
    public string ImageUrl { get; set; }
}

public class ProductUpdateDto
{
    public string Name { get; set; }
    public string Description { get; set; }
    public decimal? Price { get; set; }
    public bool? IsAvailable { get; set; }
    public string ImageUrl { get; set; }
}
```

```csharp
// DTOs/OrderDTOs.cs
namespace TinyTreats.DTO;

public class OrderDto
{
    public int Id { get; set; }
    public DateTime OrderDate { get; set; }
    public DateTime? DeliveryDate { get; set; }
    public string Status { get; set; }
    public int UserProfileId { get; set; }
    public string CustomerName { get; set; }
    public List<OrderItemDto> Items { get; set; }
    public decimal TotalAmount { get; set; }
}

public class OrderItemDto
{
    public int ProductId { get; set; }
    public string ProductName { get; set; }
    public decimal ProductPrice { get; set; }
    public int Quantity { get; set; }
    public decimal Subtotal { get; set; }
}

public class OrderCreateDto
{
    public List<OrderItemCreateDto> Items { get; set; }
}

public class OrderItemCreateDto
{
    public int ProductId { get; set; }
    public int Quantity { get; set; }
}

public class OrderStatusUpdateDto
{
    public required string Status { get; set; }
}
```

## Understanding the DTO Design

Our DTOs are designed with specific purposes:

1. **Authentication DTOs**:
   - `RegistrationDto`: For user registration with all required fields
   - `LoginDto`: For user login with email and password
   - `UserProfileDto`: For returning user profile information

2. **Role DTOs**:
   - `RoleDto`: For creating new roles
   - `UserRoleDto`: For assigning roles to users

3. **Product DTOs**:
   - `ProductDto`: For returning product information
   - `ProductCreateDto`: For creating new products
   - `ProductUpdateDto`: For updating existing products (with nullable properties)

4. **Order DTOs**:
   - `OrderDto`: For returning order information with customer and item details
   - `OrderItemDto`: For returning order item information with product details
   - `OrderCreateDto`: For creating new orders
   - `OrderItemCreateDto`: For creating order items within an order
   - `OrderStatusUpdateDto`: For updating the status of an order

## Benefits of Using DTOs

Using DTOs provides several benefits:

1. **Separation of concerns**: DTOs decouple your API contract from your database schema
2. **Security**: You can exclude sensitive data from being sent to clients
3. **Flexibility**: You can shape the data differently for different endpoints
4. **Validation**: You can apply specific validation rules for API requests
5. **Evolution**: You can evolve your API independently from your database schema

## Summary

In this chapter, we've defined all the database models and DTOs needed for our TinyTreats bakery management system:

- User-related models: `UserProfile`
- Product model: `Product`
- Order-related models: `Order` and `OrderItem`
- DTOs for authentication, roles, products, and orders

These models and DTOs provide a solid foundation for our application. In the next chapter, we'll create the database context that will manage these models and their relationships.

[Next: Creating the TinyTreatsDbContext](./tinytreats-dbcontext.md)