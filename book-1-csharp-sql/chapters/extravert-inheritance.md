# Inheritance with Plant Types

In this chapter, we'll explore inheritance in C# by creating specialized plant types for our ExtraVert Garden application.

## Learning Objectives

By the end of this chapter, you should be able to:
- Understand and implement inheritance
- Create derived classes that inherit from a base class
- Override methods in derived classes
- Understand when to use inheritance

## Inheritance Basics

Inheritance allows you to create new classes that reuse, extend, and modify the behavior defined in other classes. The class whose members are inherited is called the **base class**, and the class that inherits those members is called the **derived class**.

In C#, we use the colon (`:`) symbol to indicate inheritance. For example:

```csharp
public class DerivedClass : BaseClass
{
    // Additional members and overridden methods
}
```

## Creating Specialized Plant Types

Let's create two specialized plant types that inherit from our existing `Plant` class: `Cactus` and `Tree`.

### Creating the Cactus Plant Class

First, let's create a class for cacti. Create a new file called `Models/Cactus.cs`:

```csharp
// Models/Cactus.cs
using System;

namespace ExtraVert.Models
{
    public class Cactus : Plant
    {
        // Additional properties specific to cacti
        public bool HasSpines { get; set; }
        public string SpineLength { get; set; }
        public bool IsDesertType { get; set; }

        // Constructor
        public Cactus(string name, string species, string lightNeeds, string waterNeeds,
                          decimal price, bool hasSpines, string spineLength, bool isDesertType)
            : base(name, species, lightNeeds, waterNeeds, price)
        {
            HasSpines = hasSpines;
            SpineLength = spineLength;
            IsDesertType = isDesertType;
        }

        // Override the DisplayInfo method to include cactus-specific information
        public override void DisplayInfo()
        {
            // Call the base class DisplayInfo method
            base.DisplayInfo();

            // Add cactus-specific information
            Console.WriteLine($"Has Spines: {(HasSpines ? "Yes" : "No")}");
            Console.WriteLine($"Spine Length: {SpineLength}");
            Console.WriteLine($"Desert Type: {(IsDesertType ? "Yes" : "No")}");
        }

        // Override the GetPlantType method
        public override string GetPlantType()
        {
            return "Cactus Plant";
        }

        // Override the Water method for cacti
        public override void Water()
        {
            Console.WriteLine($"Watering {Name} very sparingly - cacti need minimal water.");
        }

        // Add a method specific to cacti
        public void HandleCarefully()
        {
            Console.WriteLine($"Handling {Name} carefully to avoid spine injuries.");
        }
    }
}
```

In this class:
- We use the `: Plant` syntax to indicate that `Cactus` inherits from `Plant`
- We add properties specific to cacti: `HasSpines`, `SpineLength`, and `IsDesertType`
- We create a constructor that takes parameters for all properties, including those from the base class
- We use the `: base(...)` syntax to call the base class constructor with the appropriate parameters
- We override the `DisplayInfo` method to include the cactus-specific information
- We call `base.DisplayInfo()` to include the information from the base class
- We override the `GetPlantType` method to return "Cactus Plant"
- We override the `Water` method to provide cactus-specific watering instructions
- We add a `HandleCarefully` method that's specific to cacti

### Creating the Tree Plant Class

Next, let's create a class for trees. Create a new file called `Models/Tree.cs`:

```csharp
// Models/Tree.cs
using System;

namespace ExtraVert.Models
{
    public class Tree : Plant
    {
        // Additional properties specific to trees
        public double HeightInFeet { get; set; }
        public bool IsDeciduous { get; set; }
        public string BarkType { get; set; }

        // Constructor
        public Tree(string name, string species, string lightNeeds, string waterNeeds,
                        decimal price, double heightInFeet, bool isDeciduous, string barkType)
            : base(name, species, lightNeeds, waterNeeds, price)
        {
            HeightInFeet = heightInFeet;
            IsDeciduous = isDeciduous;
            BarkType = barkType;
        }

        // Override the DisplayInfo method to include tree-specific information
        public override void DisplayInfo()
        {
            // Call the base class DisplayInfo method
            base.DisplayInfo();

            // Add tree-specific information
            Console.WriteLine($"Height: {HeightInFeet} feet");
            Console.WriteLine($"Deciduous: {(IsDeciduous ? "Yes" : "No")}");
            Console.WriteLine($"Bark Type: {BarkType}");
        }

        // Override the GetPlantType method
        public override string GetPlantType()
        {
            return "Tree Plant";
        }

        // Add a method specific to trees
        public void Prune()
        {
            Console.WriteLine($"Pruning {Name} to maintain its shape and health.");
        }
    }
}
```

## Updating the Plant Repository

Now let's update the `PlantRepository` class to include our new plant types:

```csharp
// Data/PlantRepository.cs
using System;
using System.Collections.Generic;
using ExtraVert.Models;

namespace ExtraVert.Data
{
    public class PlantRepository
    {
        private List<Plant> _plants = new List<Plant>();

        public List<Plant> GetAllPlants()
        {
            return _plants;
        }

        public void AddPlant(Plant plant)
        {
            _plants.Add(plant);
        }

        public List<Plant> SearchPlants(string searchTerm)
        {
            return _plants.FindAll(p =>
                p.Name.Contains(searchTerm, StringComparison.OrdinalIgnoreCase) ||
                p.Species.Contains(searchTerm, StringComparison.OrdinalIgnoreCase) ||
                p.GetPlantType().Contains(searchTerm, StringComparison.OrdinalIgnoreCase));
        }

        public void SeedData()
        {
            // Add some sample plants of different types

            // Cactus plants
            _plants.Add(new Cactus(
                "Saguaro Cactus",
                "Carnegiea gigantea",
                "Full sun",
                "Very low",
                45.99m,
                true,
                "2-3 inches",
                true));

            // Tree plants
            _plants.Add(new Tree(
                "Japanese Maple",
                "Acer palmatum",
                "Partial shade to full sun",
                "Moderate",
                89.99m,
                8.5,
                true,
                "Smooth, gray-brown"));

            // Add other plant types as needed
        }
    }
}
```

## Running the Application

Now that we've updated our application with new plant types, let's run it to see the changes:

```bash
dotnet run
```

You should see the welcome message and the main menu. You can select option 1 to view all plants, which will now display the plant type and additional information for each plant based on its specific type.

## When to Use Inheritance

Inheritance is a powerful feature of object-oriented programming, but it's not always the right tool for the job. Here are some guidelines for when to use inheritance:

1. **Use inheritance when there is an "is-a" relationship**: A cactus is a plant, a tree is a plant. This is a clear "is-a" relationship, making inheritance appropriate.

2. **Use inheritance for code reuse**: If multiple classes share common properties and behaviors, inheritance can help avoid code duplication.

3. **Use inheritance to implement polymorphism**: If you need to treat objects of different classes uniformly, inheritance can help.

## Practice Exercise

Enhance the ExtraVert Garden application by:
1. Creating a new derived class called `Flower` that inherits from `Plant`
2. Adding appropriate properties to the `Flower` class (e.g., `Color`, `Season`, `PetalSize`)
3. Implementing the necessary methods and overriding appropriate methods
4. Adding a method specific to flowers (e.g., `Bloom`)
5. Adding some flowers to the `SeedData` method
6. Running the application and verifying that the flowers are displayed correctly

## Next Steps

In the next chapter, we'll explore properties and encapsulation in C#. We'll learn how to use properties to control access to class fields and how to implement data validation.

Before moving on, make sure you're comfortable with:
- Creating and using inheritance
- Creating derived classes that inherit from a base class
- Overriding methods in derived classes
- Understanding when to use inheritance


[Next Chapter: Properties and Encapsulation](./extravert-properties.md)