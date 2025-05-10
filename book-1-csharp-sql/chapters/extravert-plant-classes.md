# Creating Plant Classes

In this chapter, we'll expand the ExtraVert Garden application by creating a hierarchy of plant classes. We'll learn about class inheritance and how to create specialized classes that inherit from a base class.

## Learning Objectives

By the end of this chapter, you should be able to:
- Understand the concept of inheritance in object-oriented programming
- Create a base class with common properties and methods
- Create derived classes that inherit from a base class
- Override methods in derived classes
- Implement class constructors with parameters

## Understanding Inheritance

Inheritance is a fundamental concept in object-oriented programming that allows you to create new classes based on existing ones. The new class (derived class) inherits the properties and methods of the existing class (base class) and can add its own properties and methods or override the inherited ones.

Inheritance helps you:
- Reuse code by sharing common functionality
- Create a hierarchy of related classes
- Model real-world relationships between objects
- Implement polymorphism (which we'll explore in later chapters)

## Planning Our Plant Class Hierarchy

For the ExtraVert Garden application, we'll create a hierarchy of plant classes:

- `Plant`: The base class with common properties and methods for all plants
- `FloweringPlant`: A derived class for plants that produce flowers
- `FoliagePlant`: A derived class for plants grown primarily for their leaves
- `SucculentPlant`: A derived class for plants that store water in their leaves or stems

Let's start by enhancing our existing `Plant` class to serve as a better base class.

## Enhancing the Plant Base Class

Open the `Models/Plant.cs` file and update it as follows:

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
        public DateTime AcquisitionDate { get; set; }

        // Constructor
        public Plant(string name, string species, string lightNeeds, string waterNeeds, decimal price)
        {
            Name = name;
            Species = species;
            LightNeeds = lightNeeds;
            WaterNeeds = waterNeeds;
            Price = price;
            AcquisitionDate = DateTime.Now;
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
        }

        public virtual string GetPlantType()
        {
            return "Generic Plant";
        }

        public virtual void Water()
        {
            Console.WriteLine($"Watering {Name} according to its {WaterNeeds} water needs.");
        }
    }
}
```

We've made the following changes to the `Plant` class:
- Added an `AcquisitionDate` property to track when the plant was acquired
- Added the `virtual` keyword to the `DisplayInfo` method, which allows derived classes to override it
- Added a virtual `GetPlantType` method that returns a string indicating the type of plant
- Added a virtual `Water` method that simulates watering the plant

The `virtual` keyword is important because it allows derived classes to provide their own implementation of these methods.

## Creating the Flowering Plant Class

Now, let's create a derived class for flowering plants. Create a new file called `Models/FloweringPlant.cs`:

```csharp
// Models/FloweringPlant.cs
using System;

namespace ExtraVert.Models
{
    public class FloweringPlant : Plant
    {
        // Additional properties specific to flowering plants
        public string FlowerColor { get; set; }
        public bool IsAnnual { get; set; }
        public string BloomingSeason { get; set; }

        // Constructor
        public FloweringPlant(string name, string species, string lightNeeds, string waterNeeds,
                             decimal price, string flowerColor, bool isAnnual, string bloomingSeason)
            : base(name, species, lightNeeds, waterNeeds, price)
        {
            FlowerColor = flowerColor;
            IsAnnual = isAnnual;
            BloomingSeason = bloomingSeason;
        }

        // Override the DisplayInfo method to include flowering plant specific information
        public override void DisplayInfo()
        {
            // Call the base class DisplayInfo method
            base.DisplayInfo();

            // Add flowering plant specific information
            Console.WriteLine($"Flower Color: {FlowerColor}");
            Console.WriteLine($"Annual: {(IsAnnual ? "Yes" : "No")}");
            Console.WriteLine($"Blooming Season: {BloomingSeason}");
        }

        // Override the GetPlantType method
        public override string GetPlantType()
        {
            return "Flowering Plant";
        }

        // Add a method specific to flowering plants
        public void Deadhead()
        {
            Console.WriteLine($"Deadheading {Name} to encourage more blooms.");
        }
    }
}
```

In this class:
- We use the `: Plant` syntax to indicate that `FloweringPlant` inherits from `Plant`
- We add properties specific to flowering plants: `FlowerColor`, `IsAnnual`, and `BloomingSeason`
- We create a constructor that takes parameters for all properties, including those from the base class
- We use the `: base(...)` syntax to call the base class constructor with the appropriate parameters
- We override the `DisplayInfo` method to include the flowering plant specific information
- We call `base.DisplayInfo()` to include the information from the base class
- We override the `GetPlantType` method to return "Flowering Plant"
- We add a `Deadhead` method that's specific to flowering plants

## Creating the Foliage Plant Class

Next, let's create a class for foliage plants. Create a new file called `Models/FoliagePlant.cs`:

```csharp
// Models/FoliagePlant.cs
using System;

namespace ExtraVert.Models
{
    public class FoliagePlant : Plant
    {
        // Additional properties specific to foliage plants
        public string LeafColor { get; set; }
        public bool IsVariegated { get; set; }
        public string LeafShape { get; set; }

        // Constructor
        public FoliagePlant(string name, string species, string lightNeeds, string waterNeeds,
                           decimal price, string leafColor, bool isVariegated, string leafShape)
            : base(name, species, lightNeeds, waterNeeds, price)
        {
            LeafColor = leafColor;
            IsVariegated = isVariegated;
            LeafShape = leafShape;
        }

        // Override the DisplayInfo method to include foliage plant specific information
        public override void DisplayInfo()
        {
            // Call the base class DisplayInfo method
            base.DisplayInfo();

            // Add foliage plant specific information
            Console.WriteLine($"Leaf Color: {LeafColor}");
            Console.WriteLine($"Variegated: {(IsVariegated ? "Yes" : "No")}");
            Console.WriteLine($"Leaf Shape: {LeafShape}");
        }

        // Override the GetPlantType method
        public override string GetPlantType()
        {
            return "Foliage Plant";
        }

        // Add a method specific to foliage plants
        public void Prune()
        {
            Console.WriteLine($"Pruning {Name} to maintain its shape and encourage bushier growth.");
        }
    }
}
```

## Creating the Succulent Plant Class

Finally, let's create a class for succulent plants. Create a new file called `Models/SucculentPlant.cs`:

```csharp
// Models/SucculentPlant.cs
using System;

namespace ExtraVert.Models
{
    public class SucculentPlant : Plant
    {
        // Additional properties specific to succulent plants
        public bool IsIndoor { get; set; }
        public string WaterStorageOrgan { get; set; }
        public bool IsToxicToPets { get; set; }

        // Constructor
        public SucculentPlant(string name, string species, string lightNeeds, string waterNeeds,
                             decimal price, bool isIndoor, string waterStorageOrgan, bool isToxicToPets)
            : base(name, species, lightNeeds, waterNeeds, price)
        {
            IsIndoor = isIndoor;
            WaterStorageOrgan = waterStorageOrgan;
            IsToxicToPets = isToxicToPets;
        }

        // Override the DisplayInfo method to include succulent plant specific information
        public override void DisplayInfo()
        {
            // Call the base class DisplayInfo method
            base.DisplayInfo();

            // Add succulent plant specific information
            Console.WriteLine($"Indoor: {(IsIndoor ? "Yes" : "No")}");
            Console.WriteLine($"Water Storage Organ: {WaterStorageOrgan}");
            Console.WriteLine($"Toxic to Pets: {(IsToxicToPets ? "Yes" : "No")}");
        }

        // Override the GetPlantType method
        public override string GetPlantType()
        {
            return "Succulent Plant";
        }

        // Override the Water method for succulents
        public override void Water()
        {
            Console.WriteLine($"Watering {Name} sparingly - succulents need less water than other plants.");
        }

        // Add a method specific to succulent plants
        public void Propagate()
        {
            Console.WriteLine($"Propagating {Name} by taking a leaf or stem cutting.");
        }
    }
}
```

In this class, we override the `Water` method to provide a succulent-specific implementation, since succulents typically need less water than other plants.

## Updating the Plant Repository

Now that we have our plant class hierarchy, let's update the `PlantRepository` class to use our new classes:

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

            // Flowering plants
            _plants.Add(new FloweringPlant(
                "Rose",
                "Rosa",
                "Full sun",
                "Moderate",
                24.99m,
                "Red",
                false,
                "Spring to Fall"));

            _plants.Add(new FloweringPlant(
                "Sunflower",
                "Helianthus annuus",
                "Full sun",
                "Moderate",
                12.99m,
                "Yellow",
                true,
                "Summer"));

            // Foliage plants
            _plants.Add(new FoliagePlant(
                "Monstera",
                "Monstera deliciosa",
                "Bright indirect",
                "Moderate",
                29.99m,
                "Green",
                false,
                "Split"));

            _plants.Add(new FoliagePlant(
                "Pothos",
                "Epipremnum aureum",
                "Low to bright indirect",
                "Low to moderate",
                15.99m,
                "Green and yellow",
                true,
                "Heart-shaped"));

            // Succulent plants
            _plants.Add(new SucculentPlant(
                "Aloe Vera",
                "Aloe barbadensis miller",
                "Bright indirect to full sun",
                "Low",
                18.99m,
                true,
                "Leaves",
                true));

            _plants.Add(new SucculentPlant(
                "Jade Plant",
                "Crassula ovata",
                "Bright indirect to full sun",
                "Low",
                22.99m,
                true,
                "Stems and leaves",
                true));
        }
    }
}
```

We've updated the `SeedData` method to create instances of our derived classes instead of the base `Plant` class. We've also added a `SearchPlants` method that allows searching for plants by name, species, or plant type.

## Updating the Program

Finally, let's update the `Program.cs` file to demonstrate the use of our plant class hierarchy:

```csharp
// Program.cs
using System;
using ExtraVert.Data;
using ExtraVert.Models;
using ExtraVert.UI;

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
                        foreach (Plant plant in plantRepo.GetAllPlants())
                        {
                            Console.WriteLine($"Plant Type: {plant.GetPlantType()}");
                            plant.DisplayInfo();
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
                                Console.WriteLine();
                            }
                        }

                        Console.WriteLine("Press any key to continue...");
                        Console.ReadKey();
                        break;

                    case 4: // Exit
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

We've updated the program to display the plant type for each plant and implemented the search functionality.

## Running the Application

Now that we've updated our application with a plant class hierarchy, let's run it to see the changes:

```bash
dotnet run
```

You should see the welcome message and the main menu. You can select option 1 to view all plants, which will now display the plant type and additional information for each plant based on its specific type.

You can also try option 3 to search for plants. Try searching for terms like "flower", "foliage", "succulent", "green", or specific plant names.

## Understanding Polymorphism

What we've implemented in this chapter is a form of polymorphism called "subtype polymorphism" or "runtime polymorphism." Polymorphism allows objects of different classes to be treated as objects of a common base class.

In our application, we're storing instances of `FloweringPlant`, `FoliagePlant`, and `SucculentPlant` in a list of `Plant` objects. When we call methods like `DisplayInfo` or `GetPlantType` on these objects, the appropriate implementation is called based on the actual type of the object, not the type of the reference.

This is a powerful feature of object-oriented programming that allows for more flexible and extensible code.

## Next Steps

In the next chapter, we'll explore inheritance further by implementing more complex relationships between our plant classes. We'll also learn about abstract classes and how they can be used to define a common interface for derived classes.

Before moving on, make sure you're comfortable with:
- Creating a base class with virtual methods
- Creating derived classes that inherit from a base class
- Overriding methods in derived classes
- Implementing constructors in derived classes that call the base class constructor
- Understanding how polymorphism works in C#

## Practice Exercise

Enhance the ExtraVert Garden application by:
1. Adding a new derived class called `HerbPlant` for culinary and medicinal herbs
2. Adding appropriate properties to the `HerbPlant` class (e.g., `CulinaryUses`, `MedicinalUses`, `Aroma`)
3. Overriding the `DisplayInfo` and `GetPlantType` methods
4. Adding a method specific to herbs (e.g., `Harvest`)
5. Adding some herb plants to the `SeedData` method
6. Running the application and verifying that the herb plants are displayed correctly


[Next Chapter: Inheritance](./extravert-inheritance.md)