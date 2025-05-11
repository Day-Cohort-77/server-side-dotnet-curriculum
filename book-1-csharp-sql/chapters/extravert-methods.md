# Methods and Behaviors

In this chapter, we'll explore methods in C#, which define the behaviors of our classes. Methods allow objects to perform actions and interact with other objects.

## Learning Objectives

By the end of this chapter, you should be able to:
- Define and call methods in C# classes
- Understand method parameters and return types
- Use method overloading to provide multiple versions of a method
- Implement optional parameters and named arguments

## Understanding Methods

Methods are blocks of code that perform specific tasks. They encapsulate functionality, promote code reuse, and help organize code into logical units. In C#, methods are defined within classes and can be called on objects of those classes.

Let's create a simple `Plant` class with methods to demonstrate various method concepts:

```csharp
// Models/Plant.cs
using System;

namespace ExtraVert.Models
{
    public class Plant
    {
        // Properties
        public string Name { get; set; }
        public string Species { get; set; }
        public string LightNeeds { get; set; }
        public string WaterNeeds { get; set; }
        public decimal Price { get; set; }
        public DateTime AcquisitionDate { get; private set; }
        public DateTime LastWateredDate { get; private set; }

        // Constructor
        public Plant(string name, string species, string lightNeeds, string waterNeeds, decimal price)
        {
            Name = name;
            Species = species;
            LightNeeds = lightNeeds;
            WaterNeeds = waterNeeds;
            Price = price;
            AcquisitionDate = DateTime.Now;
            LastWateredDate = DateTime.Now;
        }

        // Method with no parameters and no return value
        public void DisplayInfo()
        {
            Console.WriteLine($"Name: {Name}");
            Console.WriteLine($"Species: {Species}");
            Console.WriteLine($"Light Needs: {LightNeeds}");
            Console.WriteLine($"Water Needs: {WaterNeeds}");
            Console.WriteLine($"Price: ${Price}");
            Console.WriteLine($"Acquisition Date: {AcquisitionDate.ToShortDateString()}");
            Console.WriteLine($"Last Watered: {LastWateredDate.ToShortDateString()}");
        }

        // Method with no parameters but with a return value
        public string GetPlantType()
        {
            return "Basic Plant";
        }

        // Method with no parameters but with a return value
        public string GetCareInstructions()
        {
            return $"Water {Name} according to its {WaterNeeds} water needs.";
        }

        // Method with no parameters and no return value
        public void Water()
        {
            Console.WriteLine($"Watering {Name} according to its {WaterNeeds} water needs.");
            LastWateredDate = DateTime.Now;
        }

        // Method with parameters and no return value
        public void Fertilize(string fertilizerType)
        {
            Console.WriteLine($"Fertilizing {Name} with {fertilizerType}.");
        }

        // Method with optional parameters
        public void Repot(string potSize, string soilType = "All-Purpose", bool addDrainage = true)
        {
            string drainageInfo = addDrainage ? "with" : "without";
            Console.WriteLine($"Repotting {Name} into a {potSize} pot with {soilType} soil {drainageInfo} drainage layer.");
        }

        // Method with return value
        public bool NeedsWatering()
        {
            int daysSinceLastWatered = (DateTime.Now - LastWateredDate).Days;

            // Different plants have different watering needs
            switch (WaterNeeds.ToLower())
            {
                case "low":
                    return daysSinceLastWatered > 14;
                case "moderate":
                    return daysSinceLastWatered > 7;
                case "high":
                    return daysSinceLastWatered > 3;
                default:
                    return daysSinceLastWatered > 7;
            }
        }

        // Method with multiple return values using tuples
        public (bool needsWater, bool needsRepotting) AssessCareNeeds()
        {
            bool needsWater = NeedsWatering();
            int daysSinceAcquisition = (DateTime.Now - AcquisitionDate).Days;
            bool needsRepot = daysSinceAcquisition > 365; // Simplified example - repot after a year

            return (needsWater, needsRepot);
        }
    }
}
```

In this class:
- We've created a simple `Plant` class with basic properties
- We've added methods to demonstrate different method concepts
- We've used auto-implemented properties for simplicity
- We've focused on the core method concepts

## Optional Parameters and Named Arguments

Optional parameters allow you to specify default values for parameters, making them optional when calling the method. Named arguments allow you to specify arguments by parameter name rather than position.

We've seen optional parameters in the `Repot` method:

```csharp
public void Repot(string potSize, string soilType = "All-Purpose", bool addDrainage = true)
{
    string drainageInfo = addDrainage ? "with" : "without";
    Console.WriteLine($"Repotting {Name} into a {potSize} pot with {soilType} soil {drainageInfo} drainage layer.");
}
```

Here, `soilType` and `addDrainage` are optional parameters with default values.

You can call this method in several ways:

```csharp
plant.Repot("Medium"); // Uses default values for soilType and addDrainage
plant.Repot("Large", "Cactus Mix"); // Uses default value for addDrainage
plant.Repot("Small", "Orchid Mix", false); // Specifies all parameters

// Using named arguments
plant.Repot(potSize: "Medium", addDrainage: false); // Skips soilType, uses default
plant.Repot(soilType: "Peat Moss", potSize: "Large"); // Parameters in different order
```

## Next Steps

In the next chapter, we'll explore interfaces and abstraction in C#. We'll learn how to define and implement interfaces, and how to use them to create more flexible and maintainable code.

Before moving on, make sure you're comfortable with:
- Defining and calling methods in C# classes
- Understanding method parameters and return types
- Using method overloading to provide multiple versions of a method
- Implementing optional parameters and named arguments

## Practice Exercise

Enhance the ExtraVert Garden application by:
1. Adding a new method to the `Plant` class called `Prune` that takes a `percentageToPrune` parameter
2. Adding a new method to the `PlantCareService` class called `PrunePlants` that prunes all plants in a list
3. Creating a simple console application that demonstrates these new methods
4. Running the application and verifying that the new methods work correctly

[Next Chapter: Interfaces](./extravert-interfaces.md)