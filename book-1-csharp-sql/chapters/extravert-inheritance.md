# Inheritance with Plant Types

In this chapter, we'll dive deeper into inheritance in C# by exploring more advanced concepts and implementing a more sophisticated plant type hierarchy for our ExtraVert Garden application.

## Learning Objectives

By the end of this chapter, you should be able to:
- Understand and implement multi-level inheritance
- Create and use abstract classes
- Implement the "is-a" relationship through inheritance
- Apply the Liskov Substitution Principle
- Use the `protected` access modifier
- Understand when to use inheritance versus composition

## Multi-level Inheritance

So far, we've implemented single-level inheritance, where our plant types (`FloweringPlant`, `FoliagePlant`, and `SucculentPlant`) directly inherit from the `Plant` base class. However, C# also supports multi-level inheritance, where a class can inherit from a derived class.

Let's enhance our plant hierarchy by adding some specialized plant types that inherit from our existing derived classes.

### Creating a Rose Class

Let's create a specialized class for roses that inherits from `FloweringPlant`. Create a new file called `Models/Rose.cs`:

```csharp
// Models/Rose.cs
using System;

namespace ExtraVert.Models
{
    public class Rose : FloweringPlant
    {
        // Additional properties specific to roses
        public bool HasThorns { get; set; }
        public string Fragrance { get; set; }
        public string RoseType { get; set; } // e.g., Hybrid Tea, Floribunda, Climbing

        // Constructor
        public Rose(string name, string species, string lightNeeds, string waterNeeds,
                   decimal price, string flowerColor, bool isAnnual, string bloomingSeason,
                   bool hasThorns, string fragrance, string roseType)
            : base(name, species, lightNeeds, waterNeeds, price, flowerColor, isAnnual, bloomingSeason)
        {
            HasThorns = hasThorns;
            Fragrance = fragrance;
            RoseType = roseType;
        }

        // Override the DisplayInfo method to include rose-specific information
        public override void DisplayInfo()
        {
            // Call the base class DisplayInfo method
            base.DisplayInfo();

            // Add rose-specific information
            Console.WriteLine($"Has Thorns: {(HasThorns ? "Yes" : "No")}");
            Console.WriteLine($"Fragrance: {Fragrance}");
            Console.WriteLine($"Rose Type: {RoseType}");
        }

        // Override the GetPlantType method
        public override string GetPlantType()
        {
            return "Rose";
        }

        // Add a method specific to roses
        public void PruneDeadwood()
        {
            Console.WriteLine($"Pruning dead wood from {Name} to encourage healthy growth.");
        }
    }
}
```

In this class:
- We use the `: FloweringPlant` syntax to indicate that `Rose` inherits from `FloweringPlant` (which itself inherits from `Plant`)
- We add properties specific to roses: `HasThorns`, `Fragrance`, and `RoseType`
- We create a constructor that takes parameters for all properties, including those from the base classes
- We use the `: base(...)` syntax to call the `FloweringPlant` constructor with the appropriate parameters
- We override the `DisplayInfo` method to include the rose-specific information
- We call `base.DisplayInfo()` to include the information from the `FloweringPlant` class (which in turn calls the `Plant` class's `DisplayInfo` method)
- We override the `GetPlantType` method to return "Rose"
- We add a `PruneDeadwood` method that's specific to roses

This demonstrates multi-level inheritance, where `Rose` inherits from `FloweringPlant`, which inherits from `Plant`.

## Abstract Classes

An abstract class is a class that cannot be instantiated directly and is designed to be inherited by other classes. Abstract classes can contain abstract methods (methods without an implementation) that derived classes must implement.

Let's modify our plant hierarchy to use an abstract base class. Update the `Models/Plant.cs` file:

```csharp
// Models/Plant.cs
using System;

namespace ExtraVert.Models
{
    public abstract class Plant
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

        // Abstract method that derived classes must implement
        public abstract string GetPlantType();

        // Virtual method that derived classes can override
        public virtual void Water()
        {
            Console.WriteLine($"Watering {Name} according to its {WaterNeeds} water needs.");
        }

        // Abstract method for plant care instructions
        public abstract string GetCareInstructions();
    }
}
```

We've made the following changes to the `Plant` class:
- Added the `abstract` keyword to the class declaration, making it an abstract class
- Changed the `GetPlantType` method to be abstract, which means it has no implementation and derived classes must provide one
- Added a new abstract method `GetCareInstructions` that derived classes must implement

Now we need to update our derived classes to implement these abstract methods. Let's update the `FloweringPlant` class:

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

        // Implement the abstract GetPlantType method
        public override string GetPlantType()
        {
            return "Flowering Plant";
        }

        // Implement the abstract GetCareInstructions method
        public override string GetCareInstructions()
        {
            return $"Provide {LightNeeds} light. Water {WaterNeeds}. " +
                   $"Deadhead spent blooms to encourage more flowers. " +
                   $"Fertilize during {BloomingSeason} growing season.";
        }

        // Add a method specific to flowering plants
        public void Deadhead()
        {
            Console.WriteLine($"Deadheading {Name} to encourage more blooms.");
        }
    }
}
```

We need to make similar updates to the `FoliagePlant` and `SucculentPlant` classes to implement the abstract methods. Here's the updated `FoliagePlant` class:

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

        // Implement the abstract GetPlantType method
        public override string GetPlantType()
        {
            return "Foliage Plant";
        }

        // Implement the abstract GetCareInstructions method
        public override string GetCareInstructions()
        {
            return $"Provide {LightNeeds} light. Water {WaterNeeds}. " +
                   $"Mist regularly to increase humidity. " +
                   $"Wipe leaves occasionally to remove dust.";
        }

        // Add a method specific to foliage plants
        public void Prune()
        {
            Console.WriteLine($"Pruning {Name} to maintain its shape and encourage bushier growth.");
        }
    }
}
```

And here's the updated `SucculentPlant` class:

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

        // Implement the abstract GetPlantType method
        public override string GetPlantType()
        {
            return "Succulent Plant";
        }

        // Implement the abstract GetCareInstructions method
        public override string GetCareInstructions()
        {
            return $"Provide {LightNeeds} light. Water sparingly - {WaterNeeds}. " +
                   $"Allow soil to dry completely between waterings. " +
                   $"Use well-draining soil to prevent root rot.";
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

Now we need to update the `Rose` class to implement the abstract methods:

```csharp
// Models/Rose.cs
using System;

namespace ExtraVert.Models
{
    public class Rose : FloweringPlant
    {
        // Additional properties specific to roses
        public bool HasThorns { get; set; }
        public string Fragrance { get; set; }
        public string RoseType { get; set; } // e.g., Hybrid Tea, Floribunda, Climbing

        // Constructor
        public Rose(string name, string species, string lightNeeds, string waterNeeds,
                   decimal price, string flowerColor, bool isAnnual, string bloomingSeason,
                   bool hasThorns, string fragrance, string roseType)
            : base(name, species, lightNeeds, waterNeeds, price, flowerColor, isAnnual, bloomingSeason)
        {
            HasThorns = hasThorns;
            Fragrance = fragrance;
            RoseType = roseType;
        }

        // Override the DisplayInfo method to include rose-specific information
        public override void DisplayInfo()
        {
            // Call the base class DisplayInfo method
            base.DisplayInfo();

            // Add rose-specific information
            Console.WriteLine($"Has Thorns: {(HasThorns ? "Yes" : "No")}");
            Console.WriteLine($"Fragrance: {Fragrance}");
            Console.WriteLine($"Rose Type: {RoseType}");
        }

        // Override the GetPlantType method
        public override string GetPlantType()
        {
            return "Rose";
        }

        // Override the GetCareInstructions method
        public override string GetCareInstructions()
        {
            return $"Provide {LightNeeds} light. Water {WaterNeeds}. " +
                   $"Deadhead spent blooms to encourage more flowers. " +
                   $"Prune in early spring. Protect from frost in winter. " +
                   $"Watch for black spot and powdery mildew.";
        }

        // Add a method specific to roses
        public void PruneDeadwood()
        {
            Console.WriteLine($"Pruning dead wood from {Name} to encourage healthy growth.");
        }
    }
}
```

## The Protected Access Modifier

In C#, the `protected` access modifier allows a class member to be accessed within its class and by derived classes. This is useful when you want to hide implementation details from external code but allow derived classes to access them.

Let's update our `Plant` class to use the `protected` access modifier for some of its members:

```csharp
// Models/Plant.cs
using System;

namespace ExtraVert.Models
{
    public abstract class Plant
    {
        // Public properties
        public string Name { get; set; }
        public string Species { get; set; }
        public string LightNeeds { get; set; }
        public string WaterNeeds { get; set; }
        public decimal Price { get; set; }
        public DateTime AcquisitionDate { get; set; }

        // Protected field
        protected bool _needsFertilizer;

        // Constructor
        public Plant(string name, string species, string lightNeeds, string waterNeeds, decimal price)
        {
            Name = name;
            Species = species;
            LightNeeds = lightNeeds;
            WaterNeeds = waterNeeds;
            Price = price;
            AcquisitionDate = DateTime.Now;
            _needsFertilizer = true; // Most plants need fertilizer by default
        }

        // Public methods
        public virtual void DisplayInfo()
        {
            Console.WriteLine($"Name: {Name}");
            Console.WriteLine($"Species: {Species}");
            Console.WriteLine($"Light Needs: {LightNeeds}");
            Console.WriteLine($"Water Needs: {WaterNeeds}");
            Console.WriteLine($"Price: ${Price}");
            Console.WriteLine($"Acquisition Date: {AcquisitionDate.ToShortDateString()}");
            Console.WriteLine($"Needs Fertilizer: {(_needsFertilizer ? "Yes" : "No")}");
        }

        public abstract string GetPlantType();

        public virtual void Water()
        {
            Console.WriteLine($"Watering {Name} according to its {WaterNeeds} water needs.");
        }

        public abstract string GetCareInstructions();

        // Protected method
        protected void UpdateFertilizerNeeds(bool needsFertilizer)
        {
            _needsFertilizer = needsFertilizer;
            Console.WriteLine($"Updated fertilizer needs for {Name}: {(_needsFertilizer ? "Needs fertilizer" : "Does not need fertilizer")}");
        }
    }
}
```

We've added a protected field `_needsFertilizer` and a protected method `UpdateFertilizerNeeds` that can be accessed by derived classes but not by external code.

Now, let's update the `FloweringPlant` class to use these protected members:

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

            // Use the protected method from the base class
            if (isAnnual)
            {
                // Annual plants typically need more fertilizer
                UpdateFertilizerNeeds(true);
            }
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

        // Implement the abstract GetPlantType method
        public override string GetPlantType()
        {
            return "Flowering Plant";
        }

        // Implement the abstract GetCareInstructions method
        public override string GetCareInstructions()
        {
            string fertilizerInstructions = _needsFertilizer
                ? "Fertilize regularly during growing season."
                : "Minimal fertilizer needed.";

            return $"Provide {LightNeeds} light. Water {WaterNeeds}. " +
                   $"Deadhead spent blooms to encourage more flowers. " +
                   $"{fertilizerInstructions}";
        }

        // Add a method specific to flowering plants
        public void Deadhead()
        {
            Console.WriteLine($"Deadheading {Name} to encourage more blooms.");

            // After deadheading, the plant might need fertilizer
            UpdateFertilizerNeeds(true);
        }
    }
}
```

In this updated class, we're using the protected field `_needsFertilizer` and the protected method `UpdateFertilizerNeeds` from the base class.

## Updating the Program

Let's update the `Program.cs` file to demonstrate the use of our enhanced plant class hierarchy:

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

We've updated the program to display the care instructions for each plant.

## Updating the Plant Repository

Finally, let's update the `PlantRepository` class to include a `Rose` instance:

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
                "Sunflower",
                "Helianthus annuus",
                "Full sun",
                "Moderate",
                12.99m,
                "Yellow",
                true,
                "Summer"));

            // Rose (derived from FloweringPlant)
            _plants.Add(new Rose(
                "Hybrid Tea Rose",
                "Rosa 'Peace'",
                "Full sun",
                "Moderate",
                24.99m,
                "Yellow with pink edges",
                false,
                "Spring to Fall",
                true,
                "Strong, sweet",
                "Hybrid Tea"));

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

## Running the Application

Now that we've updated our application with a more sophisticated plant class hierarchy, let's run it to see the changes:

```bash
dotnet run
```

You should see the welcome message and the main menu. You can select option 1 to view all plants, which will now display the plant type, additional information, and care instructions for each plant based on its specific type.

## When to Use Inheritance

Inheritance is a powerful feature of object-oriented programming, but it's not always the right tool for the job. Here are some guidelines for when to use inheritance:

1. **Use inheritance when there is an "is-a" relationship**: A rose is a flowering plant, which is a plant. This is a clear "is-a" relationship, making inheritance appropriate.

2. **Use inheritance for code reuse**: If multiple classes share common properties and behaviors, inheritance can help avoid code duplication.

3. **Use inheritance to implement polymorphism**: If you need to treat objects of different classes uniformly, inheritance and polymorphism can help.

4. **Consider composition for "has-a" relationships**: If a class has a component rather than being a type of another class, composition (where one class contains an instance of another) might be more appropriate.

5. **Avoid deep inheritance hierarchies**: Deep inheritance hierarchies can become complex and difficult to understand. Try to keep your inheritance hierarchy shallow.

6. **Follow the Liskov Substitution Principle**: Derived classes should be substitutable for their base classes without altering the correctness of the program.

## Next Steps

In the next chapter, we'll explore properties and encapsulation in C#. We'll learn how to use properties to control access to class fields and how to implement data validation.

Before moving on, make sure you're comfortable with:
- Creating and using abstract classes
- Implementing abstract methods in derived classes
- Using the protected access modifier
- Understanding when to use inheritance
- Implementing multi-level inheritance

## Practice Exercise

Enhance the ExtraVert Garden application by:
1. Creating a new derived class called `CactusPlant` that inherits from `SucculentPlant`
2. Adding appropriate properties to the `CactusPlant` class (e.g., `HasSpines`, `GrowthForm`, `FlowerColor`)
3. Implementing the abstract methods and overriding appropriate virtual methods
4. Adding a method specific to cacti (e.g., `HandleWithCare`)
5. Adding some cactus plants to the `SeedData` method
6. Running the application and verifying that the cactus plants are displayed correctly with their care instructions

[Next Chapter: Properties and Encapsulation](./extravert-properties.md)