# Understanding Data Annotations

In this chapter, we'll explore how to use Data Annotations to enhance our Entity Framework Core models. Data Annotations provide a way to configure your model classes with attributes that influence how EF Core maps them to the database.

## Understanding Data Annotations

Data Annotations are attributes that you can apply to classes or properties to specify constraints, validation rules, and mapping details. These annotations help Entity Framework Core understand how to:

1. Map your C# classes to database tables
2. Apply constraints to columns
3. Configure relationships between tables
4. Validate data before it's saved to the database

## Common Data Annotations

Let's review some of the most commonly used Data Annotations in Entity Framework Core:

### 1. [Required]

The `[Required]` attribute specifies that a property must have a value. In the database, this translates to a `NOT NULL` constraint on the corresponding column.

```csharp
[Required]
public string CampsiteTypeName { get; set; }
```

### 2. [Key]

The `[Key]` attribute identifies a property as the entity's primary key. In our models, we're using the EF Core convention of naming our primary key property `Id`, so we don't need to explicitly use this attribute.

```csharp
[Key]
public int CampsiteId { get; set; }
```

### 3. [MaxLength] and [MinLength]

These attributes specify the maximum and minimum length for string properties:

```csharp
[MaxLength(100)]
public string Nickname { get; set; }
```

### 4. [StringLength]

Similar to `[MaxLength]`, but can also specify a minimum length:

```csharp
[StringLength(100, MinimumLength = 3)]
public string Nickname { get; set; }
```

### 5. [Range]

Specifies a range constraint for numeric values:

```csharp
[Range(1, 14)]
public int MaxReservationDays { get; set; }
```

### 6. [Column]

Specifies the database column name and can also specify the data type:

```csharp
[Column("fee_per_night", TypeName = "decimal(18,2)")]
public decimal FeePerNight { get; set; }
```

### 7. [Table]

Specifies the database table name:

```csharp
[Table("CampsiteTypes")]
public class CampsiteType
{
    // Properties
}
```

### 8. [ForeignKey]

Explicitly specifies which property is a foreign key:

```csharp
public int CampsiteTypeId { get; set; }

[ForeignKey("CampsiteTypeId")]
public CampsiteType CampsiteType { get; set; }
```

## Enhancing Our Models with Data Annotations

Let's enhance our existing models with additional Data Annotations to provide more constraints and improve our database schema.

### Updated CampsiteType Model

```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace CreekRiver.Models;

public class CampsiteType
{
    public int Id { get; set; }

    [Required]
    [MaxLength(50)]
    public string CampsiteTypeName { get; set; }

    [Range(1, 14)]
    public int MaxReservationDays { get; set; }

    [Column(TypeName = "decimal(18,2)")]
    public decimal FeePerNight { get; set; }
}
```

### Updated Campsite Model

```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace CreekRiver.Models;

public class Campsite
{
    public int Id { get; set; }

    [Required]
    [MaxLength(100)]
    public string Nickname { get; set; }

    [MaxLength(255)]
    public string ImageUrl { get; set; }

    public int CampsiteTypeId { get; set; }

    [ForeignKey("CampsiteTypeId")]
    public CampsiteType CampsiteType { get; set; }

    public List<Reservation> Reservations { get; set; }
}
```

### Updated UserProfile Model

```csharp
using System.ComponentModel.DataAnnotations;

namespace CreekRiver.Models;

public class UserProfile
{
    public int Id { get; set; }

    [Required]
    [MaxLength(50)]
    public string FirstName { get; set; }

    [Required]
    [MaxLength(50)]
    public string LastName { get; set; }

    [Required]
    [MaxLength(255)]
    [EmailAddress]
    public string Email { get; set; }

    public List<Reservation> Reservations { get; set; }
}
```

### Updated Reservation Model

```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace CreekRiver.Models;

public class Reservation
{
    public int Id { get; set; }

    public int CampsiteId { get; set; }

    [ForeignKey("CampsiteId")]
    public Campsite Campsite { get; set; }

    public int UserProfileId { get; set; }

    [ForeignKey("UserProfileId")]
    public UserProfile UserProfile { get; set; }

    [Required]
    public DateTime CheckinDate { get; set; }

    [Required]
    public DateTime CheckoutDate { get; set; }
}
```

## Data Validation with Data Annotations

Data Annotations not only help with database schema generation but also provide validation for your data. When you use these annotations, Entity Framework Core will validate the data before saving it to the database.

For example, if you try to save a `UserProfile` without an email address, EF Core will throw a validation exception because the `Email` property is marked as `[Required]`.

## Conclusion

Data Annotations provide a powerful way to configure your Entity Framework Core models. They help you define constraints, validation rules, and mapping details that influence how your C# classes are mapped to database tables.

In the next chapter, we'll create the DbContext class that will use these annotated models to set up our database.

Up Next: [DbContext to manage database operations](./creek-river-dbcontext.md)

## 🔍 Additional Materials

- [System.ComponentModel.DataAnnotations Namespace](https://docs.microsoft.com/en-us/dotnet/system.componentmodel.dataannotations)
- [EF Core - Data Annotations](https://docs.microsoft.com/en-us/ef/core/modeling/entity-properties?tabs=data-annotations)
- [Validation in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/mvc/models/validation)