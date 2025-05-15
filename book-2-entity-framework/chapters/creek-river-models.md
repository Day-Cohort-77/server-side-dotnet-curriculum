# Entity Framework Models

In this chapter, we'll create the data models for our Creek River Campground API. These models will serve two important purposes:

1. Define the C# types we'll use in our application
2. Provide Entity Framework Core with the information it needs to create the corresponding database tables

## Creating the Models

Creating the data models for this application will be almost identical to the process for other APIs you've built. The biggest difference is that our models, in addition to defining the types that we'll use in the .NET application, will also be used by Entity Framework to determine what tables to create in the database. This requires a few small additions to the code.

### 1. Create the Models Directory

First, create a `Models` folder in the project directory to organize our model classes.

### 2. Create the CampsiteType Model

Add a `CampsiteType.cs` file to the `Models` folder with the following code:

```csharp
using System.ComponentModel.DataAnnotations;

namespace CreekRiver.Models;

public class CampsiteType
{
    public int Id { get; set; }
    [Required]
    public string CampsiteTypeName { get; set; }
    public int MaxReservationDays { get; set; }
    public decimal FeePerNight { get; set; }
}
```

### 3. Create the Campsite Model

Add a `Campsite.cs` file to the `Models` folder with the following code:

```csharp
using System.ComponentModel.DataAnnotations;

namespace CreekRiver.Models;

public class Campsite
{
    public int Id { get; set; }
    [Required]
    public string Nickname { get; set; }
    public string ImageUrl { get; set; }
    public int CampsiteTypeId { get; set; }
    public CampsiteType CampsiteType { get; set; }
    public List<Reservation> Reservations { get; set; }
}
```

### 4. Create the Reservation Model

Add a `Reservation.cs` file to the `Models` folder with the following code:

```csharp
namespace CreekRiver.Models;

public class Reservation
{
    public int Id { get; set; }
    public int CampsiteId { get; set; }
    public Campsite Campsite { get; set; }
    public int UserProfileId { get; set; }
    public UserProfile UserProfile { get; set; }
    public DateTime CheckinDate { get; set; }
    public DateTime CheckoutDate { get; set; }
}
```

### 5. Create the UserProfile Model

Add a `UserProfile.cs` file to the `Models` folder with the following code:

```csharp
using System.ComponentModel.DataAnnotations;

namespace CreekRiver.Models;

public class UserProfile
{
    public int Id { get; set; }
    [Required]
    public string FirstName { get; set; }
    [Required]
    public string LastName { get; set; }
    [Required]
    public string Email { get; set; }

    public List<Reservation> Reservations { get; set; }
}
```

## Understanding Entity Framework Conventions

Let's take a moment to understand some important EF Core conventions used in our models:

1. **Primary Keys**: EF Core will automatically make a property called `Id` in C# into the `PRIMARY KEY` column for the corresponding SQL database table.

2. **Required Attributes**: The `[Required]` attribute is used to specify that a property must have a value. In the database, this translates to a `NOT NULL` constraint on the corresponding column.

3. **Foreign Keys**: EF Core will notice that properties like `CampsiteTypeId` have a name pattern of `{EntityName}Id` and can infer that this is a foreign key to the `CampsiteType` entity. It will create a foreign key constraint on this column when it creates the database.

4. **Navigation Properties**: Properties like `CampsiteType` in the `Campsite` class and `Reservations` in the `CampsiteType` class are called navigation properties. They allow us to navigate between related entities in our code. EF Core uses these to establish relationships between tables.

In the next chapter, we'll create the DbContext class that will allow our application to interact with the database.

Up Next: [Creating models with DataAnnotations](./creek-river-data-annotations.md)

## 🔍 Additional Materials

- [Entity Framework Core - Entity Properties](https://docs.microsoft.com/en-us/ef/core/modeling/entity-properties)
- [C# Attributes](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/reflection-and-attributes/)
- [C# Reference Types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/reference-types)

## 🔍 Additional Materials

- [Entity Framework Core - Entity Properties](https://docs.microsoft.com/en-us/ef/core/modeling/entity-properties)
- [C# Attributes](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/reflection-and-attributes/)
- [C# Reference Types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/reference-types)