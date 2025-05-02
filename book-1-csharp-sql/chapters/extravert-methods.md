# Methods and Behaviors

In this chapter, we'll explore methods in C#, which define the behaviors of our classes. Methods allow objects to perform actions and interact with other objects.

## Learning Objectives

By the end of this chapter, you should be able to:
- Define and call methods in C# classes
- Understand method parameters and return types
- Use method overloading to provide multiple versions of a method
- Implement optional parameters and named arguments
- Create extension methods
- Understand the difference between instance methods and static methods
- Use expression-bodied methods for concise implementations

## Understanding Methods

Methods are blocks of code that perform specific tasks. They encapsulate functionality, promote code reuse, and help organize code into logical units. In C#, methods are defined within classes and can be called on objects of those classes.

Let's enhance our `Plant` class with additional methods to demonstrate various method concepts:

```csharp
// Models/Plant.cs
using System;

namespace ExtraVert.Models
{
    public abstract class Plant
    {
        // Private fields
        private string _name;
        private string _species;
        private string _lightNeeds;
        private string _waterNeeds;
        private decimal _price;
        private DateTime _acquisitionDate;
        private bool _needsFertilizer;
        private DateTime _lastWateredDate;
        private DateTime _lastFertilizedDate;

        // Properties
        public string Name
        {
            get { return _name; }
            set
            {
                if (string.IsNullOrWhiteSpace(value))
                {
                    throw new ArgumentException("Name cannot be empty or whitespace.");
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
                    throw new ArgumentException("Species cannot be empty or whitespace.");
                }
                _species = value;
            }
        }

        public string LightNeeds
        {
            get { return _lightNeeds; }
            set { _lightNeeds = value; }
        }

        public string WaterNeeds
        {
            get { return _waterNeeds; }
            set { _waterNeeds = value; }
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

        public DateTime AcquisitionDate
        {
            get { return _acquisitionDate; }
        }

        public bool NeedsFertilizer
        {
            get { return _needsFertilizer; }
            protected set { _needsFertilizer = value; }
        }

        public int DaysSinceAcquisition
        {
            get { return (DateTime.Now - _acquisitionDate).Days; }
        }

        public DateTime LastWateredDate
        {
            get { return _lastWateredDate; }
            private set { _lastWateredDate = value; }
        }

        public DateTime LastFertilizedDate
        {
            get { return _lastFertilizedDate; }
            private set { _lastFertilizedDate = value; }
        }

        public int DaysSinceLastWatered
        {
            get { return (DateTime.Now - _lastWateredDate).Days; }
        }

        public int DaysSinceLastFertilized
        {
            get { return (DateTime.Now - _lastFertilizedDate).Days; }
        }

        // Constructor
        public Plant(string name, string species, string lightNeeds, string waterNeeds, decimal price)
        {
            Name = name;
            Species = species;
            LightNeeds = lightNeeds;
            WaterNeeds = waterNeeds;
            Price = price;
            _acquisitionDate = DateTime.Now;
            _needsFertilizer = true;
            _lastWateredDate = DateTime.Now;
            _lastFertilizedDate = DateTime.Now.AddDays(-30); // Assume last fertilized a month ago
        }

        // Methods
        public virtual void DisplayInfo()
        {
            Console.WriteLine($"Name: {Name}");
            Console.WriteLine($"Species: {Species}");
            Console.WriteLine($"Light Needs: {LightNeeds}");
            Console.WriteLine($"Water Needs: {WaterNeeds}");
            Console.WriteLine($"Price: ${Price}");
            Console.WriteLine($"Acquisition Date: {AcquisitionDate.ToShortDateString()}");
            Console.WriteLine($"Days Since Acquisition: {DaysSinceAcquisition}");
            Console.WriteLine($"Last Watered: {LastWateredDate.ToShortDateString()} ({DaysSinceLastWatered} days ago)");
            Console.WriteLine($"Last Fertilized: {LastFertilizedDate.ToShortDateString()} ({DaysSinceLastFertilized} days ago)");
            Console.WriteLine($"Needs Fertilizer: {(NeedsFertilizer ? "Yes" : "No")}");
        }

        public abstract string GetPlantType();

        public abstract string GetCareInstructions();

        // Method with no parameters and no return value
        public virtual void Water()
        {
            Console.WriteLine($"Watering {Name} according to its {WaterNeeds} water needs.");
            LastWateredDate = DateTime.Now;
        }

        // Method with parameters and no return value
        public virtual void Fertilize(string fertilizerType)
        {
            Console.WriteLine($"Fertilizing {Name} with {fertilizerType}.");
            LastFertilizedDate = DateTime.Now;
            NeedsFertilizer = false;
        }

        // Method overloading - same method name, different parameters
        public virtual void Fertilize(string fertilizerType, int amount)
        {
            Console.WriteLine($"Fertilizing {Name} with {amount} grams of {fertilizerType}.");
            LastFertilizedDate = DateTime.Now;
            NeedsFertilizer = false;
        }

        // Method with optional parameters
        public virtual void Repot(string potSize, string soilType = "All-Purpose", bool addDrainage = true)
        {
            string drainageInfo = addDrainage ? "with" : "without";
            Console.WriteLine($"Repotting {Name} into a {potSize} pot with {soilType} soil {drainageInfo} drainage layer.");
        }

        // Method with return value
        public virtual bool NeedsWatering()
        {
            // Different plants have different watering needs
            // This is a simplified example - in reality, this would be more complex
            switch (WaterNeeds.ToLower())
            {
                case "low":
                    return DaysSinceLastWatered > 14;
                case "moderate":
                    return DaysSinceLastWatered > 7;
                case "high":
                    return DaysSinceLastWatered > 3;
                default:
                    return DaysSinceLastWatered > 7;
            }
        }

        // Method with multiple return values using tuples
        public virtual (bool needsWater, bool needsFertilizer, bool needsRepotting) AssessCareNeeds()
        {
            bool needsWater = NeedsWatering();
            bool needsFert = DaysSinceLastFertilized > 30;
            bool needsRepot = DaysSinceAcquisition > 365; // Simplified - assume repotting needed after a year

            return (needsWater, needsFert, needsRepot);
        }

        // Expression-bodied method
        public virtual string GetAge() => $"{DaysSinceAcquisition} days";

        // Static method
        public static Plant GetMoreExpensivePlant(Plant plant1, Plant plant2)
        {
            return plant1.Price > plant2.Price ? plant1 : plant2;
        }
    }
}
```

In this updated class:
- We've added new fields and properties for tracking watering and fertilizing
- We've added various methods to demonstrate different method concepts
- We've updated the `DisplayInfo` method to show the new information

## Method Parameters

Methods can take parameters, which are values passed to the method when it's called. Parameters allow methods to operate on different data each time they're called.

### Parameter Types

C# supports several types of parameters:

1. **Value parameters**: The default parameter type, where a copy of the value is passed to the method
2. **Reference parameters** (`ref`): A reference to the variable is passed, allowing the method to modify the original variable
3. **Output parameters** (`out`): Similar to reference parameters, but the method must assign a value to the parameter
4. **Parameter arrays** (`params`): Allows a variable number of arguments to be passed to a method

Let's create a new class called `PlantCareService` that demonstrates these parameter types:

```csharp
// Models/PlantCareService.cs
using System;
using System.Collections.Generic;

namespace ExtraVert.Models
{
    public class PlantCareService
    {
        // Method with value parameters
        public void WaterPlants(List<Plant> plants)
        {
            Console.WriteLine("Watering all plants...");
            foreach (Plant plant in plants)
            {
                if (plant.NeedsWatering())
                {
                    plant.Water();
                }
                else
                {
                    Console.WriteLine($"{plant.Name} doesn't need watering yet.");
                }
            }
        }

        // Method with reference parameter
        public void AdjustPrice(ref decimal price, decimal percentageChange)
        {
            price = price * (1 + percentageChange / 100);
        }

        // Method with output parameter
        public bool TryParsePlantType(string input, out string plantType)
        {
            input = input.ToLower().Trim();

            if (input.Contains("flower") || input.Contains("bloom"))
            {
                plantType = "Flowering Plant";
                return true;
            }
            else if (input.Contains("foliage") || input.Contains("leaf"))
            {
                plantType = "Foliage Plant";
                return true;
            }
            else if (input.Contains("succulent") || input.Contains("cactus"))
            {
                plantType = "Succulent Plant";
                return true;
            }
            else
            {
                plantType = "Unknown";
                return false;
            }
        }

        // Method with params parameter
        public void FertilizePlants(string fertilizerType, params Plant[] plants)
        {
            Console.WriteLine($"Fertilizing plants with {fertilizerType}...");
            foreach (Plant plant in plants)
            {
                plant.Fertilize(fertilizerType);
            }
        }

        // Method with multiple parameters of different types
        public void ScheduleCare(Plant plant, DateTime wateringDate, DateTime fertilizingDate, bool needsRepotting)
        {
            Console.WriteLine($"Care schedule for {plant.Name}:");
            Console.WriteLine($"Next watering: {wateringDate.ToShortDateString()}");
            Console.WriteLine($"Next fertilizing: {fertilizingDate.ToShortDateString()}");
            Console.WriteLine($"Needs repotting: {(needsRepotting ? "Yes" : "No")}");
        }
    }
}
```

In this class:
- `WaterPlants` takes a value parameter (a list of plants)
- `AdjustPrice` takes a reference parameter (`ref decimal price`), allowing it to modify the original variable
- `TryParsePlantType` takes an output parameter (`out string plantType`), which it must assign a value to
- `FertilizePlants` takes a params parameter (`params Plant[] plants`), allowing it to accept a variable number of Plant arguments
- `ScheduleCare` takes multiple parameters of different types

## Method Overloading

Method overloading allows you to define multiple methods with the same name but different parameter lists. The compiler determines which method to call based on the arguments provided.

We've already seen method overloading in the `Plant` class with the two `Fertilize` methods:

```csharp
public virtual void Fertilize(string fertilizerType)
{
    Console.WriteLine($"Fertilizing {Name} with {fertilizerType}.");
    LastFertilizedDate = DateTime.Now;
    NeedsFertilizer = false;
}

public virtual void Fertilize(string fertilizerType, int amount)
{
    Console.WriteLine($"Fertilizing {Name} with {amount} grams of {fertilizerType}.");
    LastFertilizedDate = DateTime.Now;
    NeedsFertilizer = false;
}
```

The first method takes a single parameter (the fertilizer type), while the second takes two parameters (the fertilizer type and the amount).

## Optional Parameters and Named Arguments

Optional parameters allow you to specify default values for parameters, making them optional when calling the method. Named arguments allow you to specify arguments by parameter name rather than position.

We've seen optional parameters in the `Repot` method:

```csharp
public virtual void Repot(string potSize, string soilType = "All-Purpose", bool addDrainage = true)
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

## Extension Methods

Extension methods allow you to add methods to existing types without modifying the original type. They are defined as static methods in static classes, with the first parameter having the `this` keyword before the type.

Let's create a static class with extension methods for the `Plant` class:

```csharp
// Extensions/PlantExtensions.cs
using System;
using ExtraVert.Models;

namespace ExtraVert.Extensions
{
    public static class PlantExtensions
    {
        // Extension method to check if a plant is expensive
        public static bool IsExpensive(this Plant plant, decimal threshold = 25.0m)
        {
            return plant.Price > threshold;
        }

        // Extension method to get a formatted price string
        public static string GetFormattedPrice(this Plant plant)
        {
            return $"${plant.Price:F2}";
        }

        // Extension method to get a summary of the plant
        public static string GetSummary(this Plant plant)
        {
            return $"{plant.Name} ({plant.GetPlantType()}) - {plant.GetFormattedPrice()}";
        }
    }
}
```

To use these extension methods, you need to include the namespace where they're defined:

```csharp
using ExtraVert.Extensions;

// ...

Plant plant = new FloweringPlant("Rose", "Rosa", "Full sun", "Moderate", 29.99m, "Red", false, "Spring to Fall");

bool isExpensive = plant.IsExpensive(); // Extension method
string formattedPrice = plant.GetFormattedPrice(); // Extension method
string summary = plant.GetSummary(); // Extension method
```

## Instance Methods vs. Static Methods

Instance methods are called on an instance of a class, while static methods are called on the class itself.

We've seen instance methods throughout our classes, such as `Water`, `Fertilize`, and `DisplayInfo`. These methods operate on a specific instance of a class and can access instance fields and properties.

We've also seen a static method in the `Plant` class:

```csharp
public static Plant GetMoreExpensivePlant(Plant plant1, Plant plant2)
{
    return plant1.Price > plant2.Price ? plant1 : plant2;
}
```

This method is called on the class itself, not on an instance:

```csharp
Plant moreExpensivePlant = Plant.GetMoreExpensivePlant(plant1, plant2);
```

Static methods cannot access instance fields or properties directly, but they can operate on instances passed as parameters.

## Expression-Bodied Methods

Expression-bodied methods provide a concise syntax for methods that consist of a single expression. We've seen an example in the `Plant` class:

```csharp
public virtual string GetAge() => $"{DaysSinceAcquisition} days";
```

This is equivalent to:

```csharp
public virtual string GetAge()
{
    return $"{DaysSinceAcquisition} days";
}
```

## Updating the Program

Let's update the `Program.cs` file to demonstrate the use of our enhanced methods:

```csharp
// Program.cs
using System;
using ExtraVert.Data;
using ExtraVert.Models;
using ExtraVert.UI;
using ExtraVert.Extensions;

namespace ExtraVert
{
    class Program
    {
        static void Main(string[] args)
        {
            // Initialize our repository and seed it with data
            PlantRepository plantRepo = new PlantRepository();
            plantRepo.SeedData();

            // Create our menu
            Menu menu = new Menu();

            // Create a PlantStats object
            PlantStats stats = new PlantStats(plantRepo.GetAllPlants());

            // Create an Inventory object
            Inventory inventory = new Inventory();

            // Create a PlantCareService
            PlantCareService careService = new PlantCareService();

            // Add all plants from the repository to the inventory
            foreach (Plant plant in plantRepo.GetAllPlants())
            {
                inventory.AddPlant(plant);
            }

            bool running = true;
            while (running)
            {
                Console.Clear();
                Console.WriteLine("Welcome to ExtraVert Plant Nursery Management System!");
                Console.WriteLine("------------------------------------------------");
                Console.WriteLine();

                menu.ShowMainMenu();
                int choice = menu.GetMenuChoice();

                switch (choice)
                {
                    case 1: // View All Plants
                        Console.Clear();
                        Console.WriteLine("All Plants:");
                        Console.WriteLine("----------");
                        foreach (Plant plant in inventory.GetAllPlants())
                        {
                            Console.WriteLine($"Plant Type: {plant.GetPlantType()}");
                            plant.DisplayInfo();
                            Console.WriteLine("Care Instructions:");
                            Console.WriteLine(plant.GetCareInstructions());
                            Console.WriteLine();
                        }
                        Console.WriteLine("Press any key to continue...");
                        Console.ReadKey();
                        break;

                    case 2: // Add a New Plant
                        Console.WriteLine("This feature will be implemented in a future chapter.");
                        Console.WriteLine("Press any key to continue...");
                        Console.ReadKey();
                        break;

                    case 3: // Search for Plants
                        Console.Clear();
                        Console.Write("Enter search term: ");
                        string searchTerm = Console.ReadLine();

                        var searchResults = plantRepo.SearchPlants(searchTerm);

                        Console.WriteLine($"\nSearch Results for '{searchTerm}':");
                        Console.WriteLine("-------------------------");

                        if (searchResults.Count == 0)
                        {
                            Console.WriteLine("No plants found matching your search term.");
                        }
                        else
                        {
                            foreach (Plant plant in searchResults)
                            {
                                Console.WriteLine($"Plant Type: {plant.GetPlantType()}");
                                plant.DisplayInfo();
                                Console.WriteLine("Care Instructions:");
                                Console.WriteLine(plant.GetCareInstructions());
                                Console.WriteLine();
                            }
                        }

                        Console.WriteLine("Press any key to continue...");
                        Console.ReadKey();
                        break;

                    case 4: // View Inventory Stats
                        Console.Clear();
                        stats.DisplayStats();
                        Console.WriteLine();
                        Console.WriteLine($"Inventory Total Value: ${inventory.TotalValue}");
                        Console.WriteLine($"Inventory Count: {inventory.Count}");
                        Console.WriteLine();
                        Console.WriteLine("Press any key to continue...");
                        Console.ReadKey();
                        break;

                    case 5: // Perform Plant Care
                        Console.Clear();
                        Console.WriteLine("Plant Care Options:");
                        Console.WriteLine("1. Water all plants that need watering");
                        Console.WriteLine("2. Fertilize all plants");
                        Console.WriteLine("3. Assess care needs for all plants");
                        Console.WriteLine("4. Back to main menu");
                        Console.Write("Enter your choice (1-4): ");

                        if (int.TryParse(Console.ReadLine(), out int careChoice))
                        {
                            switch (careChoice)
                            {
                                case 1:
                                    Console.Clear();
                                    careService.WaterPlants(inventory.GetAllPlants());
                                    break;

                                case 2:
                                    Console.Clear();
                                    Console.Write("Enter fertilizer type: ");
                                    string fertilizerType = Console.ReadLine();
                                    careService.FertilizePlants(fertilizerType, inventory.GetAllPlants().ToArray());
                                    break;

                                case 3:
                                    Console.Clear();
                                    Console.WriteLine("Care Needs Assessment:");
                                    Console.WriteLine("---------------------");
                                    foreach (Plant plant in inventory.GetAllPlants())
                                    {
                                        var (needsWater, needsFertilizer, needsRepotting) = plant.AssessCareNeeds();
                                        Console.WriteLine(plant.GetSummary()); // Using extension method
                                        Console.WriteLine($"Needs watering: {(needsWater ? "Yes" : "No")}");
                                        Console.WriteLine($"Needs fertilizing: {(needsFertilizer ? "Yes" : "No")}");
                                        Console.WriteLine($"Needs repotting: {(needsRepotting ? "Yes" : "No")}");
                                        Console.WriteLine();
                                    }
                                    break;

                                case 4:
                                    // Back to main menu
                                    break;

                                default:
                                    Console.WriteLine("Invalid choice.");
                                    break;
                            }
                        }
                        else
                        {
                            Console.WriteLine("Invalid choice.");
                        }

                        Console.WriteLine("Press any key to continue...");
                        Console.ReadKey();
                        break;

                    case 6: // Exit
                        running = false;
                        break;

                    default:
                        Console.WriteLine("Invalid choice. Please try again.");
                        Console.WriteLine("Press any key to continue...");
                        Console.ReadKey();
                        break;
                }
            }

            Console.WriteLine("Thank you for using ExtraVert Plant Nursery Management System!");
            Console.WriteLine("Press any key to exit...");
            Console.ReadKey();
        }
    }
}
```

We've updated the program to include a new "Perform Plant Care" option that demonstrates the use of our enhanced methods.

## Updating the Menu

Let's update the `Menu` class to include the new "Perform Plant Care" option:

```csharp
// UI/Menu.cs
using System;

namespace ExtraVert.UI
{
    public class Menu
    {
        public void ShowMainMenu()
        {
            Console.WriteLine("Main Menu:");
            Console.WriteLine("1. View All Plants");
            Console.WriteLine("2. Add a New Plant");
            Console.WriteLine("3. Search for Plants");
            Console.WriteLine("4. View Inventory Stats");
            Console.WriteLine("5. Perform Plant Care");
            Console.WriteLine("6. Exit");
            Console.Write("Enter your choice (1-6): ");
        }

        public int GetMenuChoice()
        {
            string input = Console.ReadLine();
            if (int.TryParse(input, out int choice))
            {
                return choice;
            }
            return 0; // Invalid choice
        }
    }
}
```

## Method Guidelines

Here are some guidelines for using methods effectively:

1. **Keep methods focused on a single task**: Each method should have a clear, specific purpose.

2. **Keep methods short**: Long methods are harder to understand and maintain. Consider breaking them into smaller, more focused methods.

3. **Use meaningful method names**: Method names should clearly indicate what the method does.

4. **Use verb phrases for method names**: Method names should typically start with a verb (e.g., `GetPlantType`, `WaterPlants`).

5. **Be consistent with parameter order**: Use a consistent order for parameters across similar methods.

6. **Limit the number of parameters**: Methods with many parameters can be hard to use. Consider using a parameter object or breaking the method into smaller methods.

7. **Use method overloading judiciously**: Overloaded methods should be clearly related and perform similar tasks.

8. **Document methods with comments**: Use comments to explain what a method does, what parameters it takes, and what it returns.

## Next Steps

In the next chapter, we'll explore interfaces and abstraction in C#. We'll learn how to define and implement interfaces, and how to use them to create more flexible and maintainable code.

Before moving on, make sure you're comfortable with:
- Defining and calling methods in C# classes
- Understanding method parameters and return types
- Using method overloading to provide multiple versions of a method
- Implementing optional parameters and named arguments
- Creating extension methods
- Understanding the difference between instance methods and static methods
- Using expression-bodied methods for concise implementations

## Practice Exercise

Enhance the ExtraVert Garden application by:
1. Adding a new method to the `Plant` class called `Prune` that takes a `percentageToPrune` parameter
2. Adding a new method to the `PlantCareService` class called `PrunePlants` that prunes all plants in a list
3. Adding a new extension method to the `PlantExtensions` class called `GetTimeUntilNextWatering` that calculates how many days until a plant needs to be watered again
4. Updating the "Perform Plant Care" option in the program to include a "Prune all plants" option
5. Running the application and verifying that the new methods work correctly