# Properties and Encapsulation

In this chapter, we'll explore properties and encapsulation in C#, building on our existing ExtraVert Garden application. Properties provide a way to control access to class fields, while encapsulation is the principle of hiding implementation details and exposing only what's necessary.

## Learning Objectives

By the end of this chapter, you should be able to:
- Understand the concept of encapsulation
- Create and use properties with getters and setters
- Create read-only properties
- Implement basic property validation
- Refactor existing code to improve encapsulation

## Understanding Encapsulation

Encapsulation is one of the fundamental principles of object-oriented programming. It refers to bundling data and the methods that operate on that data within a single unit (a class), and restricting access to some of the object's components.

Encapsulation helps to:
- Hide implementation details
- Protect data from unauthorized access
- Prevent data corruption
- Make code more maintainable and flexible

In C#, encapsulation is typically achieved through access modifiers (`public`, `private`, etc.) and properties.

## Fields vs. Properties

In C#, there are two ways to store data in a class:

1. **Fields**: Variables declared directly in a class
2. **Properties**: Member-like entities that provide access to fields

Fields are typically declared as private to hide them from external code, while properties provide a public interface to access and modify those fields.

Let's look at a simple example:

### No Encapsulation

```csharp
// Without properties (poor encapsulation)
public class Plant
{
    // Public fields can be accessed and modified directly
    public string Name;
    public decimal Price;
}
```

### Fields With Properties for Encapsulation

```cs
// With properties (good encapsulation)
public class Plant
{
    // Private fields - hidden from outside
    private string _name;
    private decimal _price;

    // Public properties - controlled access
    public string Name
    {
        get { return _name; }
        set { _name = value; }
    }

    public decimal Price
    {
        get { return _price; }
        set { _price = value; }
    }
}
```

## Refactoring Our Plant Class

In the previous chapter, we created a `Plant` class with auto-implemented properties. Let's refactor it to use explicit backing fields and property accessors for better encapsulation:

```csharp
// Models/Plant.cs
using System;

namespace ExtraVert.Models
{
    public class Plant
    {
        // Private backing fields
        private string _name;
        private string _species;
        private decimal _price;
        private DateTime _acquisitionDate;

        // Properties with getters and setters
        public string Name
        {
            get { return _name; }
            set
            {
                if (string.IsNullOrWhiteSpace(value))
                {
                    throw new ArgumentException("Name cannot be empty.");
                }
                _name = value;
            }
        }

        public string Species
        {
            get { return _species; }
            set
            {
                if (string.IsNullOrWhiteSpace(value))
                {
                    throw new ArgumentException("Species cannot be empty.");
                }
                _species = value;
            }
        }

        public decimal Price
        {
            get { return _price; }
            set
            {
                if (value < 0)
                {
                    throw new ArgumentException("Price cannot be negative.");
                }
                _price = value;
            }
        }

        // Read-only property
        public DateTime AcquisitionDate
        {
            get { return _acquisitionDate; }
            // No setter - can only be set in constructor
        }

        // Constructor
        public Plant(string name, string species, string lightNeeds, string waterNeeds, decimal price)
        {
            // Use properties to ensure validation
            Name = name;
            Species = species;
            LightNeeds = lightNeeds;
            WaterNeeds = waterNeeds;
            Price = price;
            _acquisitionDate = DateTime.Now; // Set directly since it's private
        }

        // Add some methods, or behaviors, to each plant
        public void DisplayInfo()
        {
            Console.WriteLine($"Name: {Name}");
            Console.WriteLine($"Species: {Species}");
            Console.WriteLine($"Price: ${Price}");
            Console.WriteLine($"Acquisition Date: {AcquisitionDate.ToShortDateString()}");
        }

        public string GetPlantType()
        {
            return "Generic Plant";
        }

        public void Water()
        {
            Console.WriteLine($"Watering {Name} according to its needs: {WaterNeeds}");
        }
    }
}
```

In this refactored class:
- We've replaced auto-implemented properties with explicit backing fields and property accessors
- We've added validation in the setters for `Name`, `Species`, and `Price`
- We've made `AcquisitionDate` a read-only property that can only be set in the constructor

## Auto-Implemented Properties

In the previous example, we used explicit backing fields and property accessors. However, C# provides a shorthand syntax for properties that don't require custom logic. These are called auto-implemented properties:

```csharp
// Regular property with backing field
private string _name;
public string Name
{
    get { return _name; }
    set { _name = value; }
}

// Auto-implemented property (equivalent)
public string Name { get; set; }
```

The compiler automatically creates a private, anonymous backing field that can only be accessed through the property's get and set accessors.

Auto-implemented properties are great for simple cases, but when you need validation or custom logic, you should use explicit backing fields and property accessors.

## Updating Our Derived Classes

Now that we've refactored our `Plant` class, we need to update our derived classes (`Cactus` and `Tree`) to work with the new implementation. Let's start with the `Cactus` class:

```csharp
// Models/Cactus.cs
using System;

namespace ExtraVert.Models
{
    public class Cactus : Plant
    {
        // Private backing fields
        private bool _hasSpines;
        private string _spineLength;
        private bool _isDesertType;

        // Properties with getters and setters
        public bool HasSpines
        {
            get { return _hasSpines; }
            set { _hasSpines = value; }
        }

        public string SpineLength
        {
            get { return _spineLength; }
            set { _spineLength = value; }
        }

        public bool IsDesertType
        {
            get { return _isDesertType; }
            set { _isDesertType = value; }
        }

        // Constructor
        public Cactus(string name, string species, string lightNeeds, string waterNeeds,
                      decimal price, bool hasSpines, string spineLength, bool isDesertType)
            : base(name, species, lightNeeds, waterNeeds, price)
        {
            HasSpines = hasSpines;
            SpineLength = spineLength;
            IsDesertType = isDesertType;
        }

        // Method specific to cacti
        public void HandleCarefully()
        {
            Console.WriteLine($"Handling {Name} carefully to avoid spine injuries.");
        }
    }
}
```

## Calculated Properties

Properties can also calculate values on-the-fly without storing the value in a field:

```csharp
// Add to Plant class
public int DaysSinceAcquisition
{
    get { return (DateTime.Now - AcquisitionDate).Days; }
}
```

This property calculates the number of days since the plant was acquired each time it's accessed.

## Read-Only Properties

You can create read-only properties by omitting the `set` accessor:

```csharp
// Read-only property
public DateTime AcquisitionDate
{
    get { return _acquisitionDate; }
    // No setter - can only be set in constructor
}
```

## Property Validation

Properties allow us to validate data before it's stored. We've already added validation to our `Plant` class for the `Name`, `Species`, and `Price` properties. Let's add more validation to our `Cactus` class:

```csharp
public string SpineLength
{
    get { return _spineLength; }
    set
    {
        if (HasSpines && string.IsNullOrWhiteSpace(value))
        {
            throw new ArgumentException("Spine length must be specified for cacti with spines.");
        }
        _spineLength = value;
    }
}
```

Now our property validates that spine length is specified for cacti with spines.

## Practice Exercise: Enhancing the Plant Class

Enhance the `Plant` class by:

1. Adding a new property called `LastWateredDate` with a private setter
2. Modifying the `Water()` method to update the `LastWateredDate` property to the current date
3. Adding a calculated property called `DaysSinceLastWatered` that returns the number of days since the plant was last watered
4. Updating the `DisplayInfo()` method to show the last watered date and days since last watered


## Summary

Properties are a powerful feature in C# that help implement encapsulation by:
- Hiding implementation details (private fields)
- Providing controlled access to data
- Allowing validation of data
- Supporting calculated values
- Enabling read-only access when needed

By refactoring our `Plant` class to use properties with explicit backing fields, we've improved encapsulation and added validation to ensure data integrity.

In the next chapter, we'll explore methods and behaviors in more detail, building on our understanding of properties and encapsulation.

[Next Chapter: Methods and Behaviors](./extravert-methods.md)