# Properties and Encapsulation

In this chapter, we'll explore properties and encapsulation in C#. Properties provide a way to control access to class fields, while encapsulation is the principle of hiding implementation details and exposing only what's necessary.

## Learning Objectives

By the end of this chapter, you should be able to:
- Understand the concept of encapsulation
- Create and use properties with getters and setters
- Implement auto-implemented properties
- Create read-only and write-only properties
- Implement property validation
- Use expression-bodied properties
- Understand property accessors and their access modifiers

## Understanding Encapsulation

Encapsulation is one of the four fundamental principles of object-oriented programming (along with inheritance, polymorphism, and abstraction). It refers to the bundling of data and methods that operate on that data within a single unit (a class), and restricting access to some of the object's components.

Encapsulation helps to:
- Hide implementation details
- Protect data from unauthorized access
- Prevent data corruption
- Make code more maintainable and flexible

In C#, encapsulation is typically achieved through the use of access modifiers (`public`, `private`, `protected`, etc.) and properties.

## Fields vs. Properties

In C#, there are two ways to store data in a class:

1. **Fields**: Variables declared directly in a class
2. **Properties**: Member-like entities that provide access to fields

Fields are typically declared as private or protected to hide them from external code, while properties provide a public interface to access and modify those fields.

Let's enhance our `Plant` class to better demonstrate encapsulation and properties:

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

        // Properties with validation
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

        // Read-only property
        public DateTime AcquisitionDate
        {
            get { return _acquisitionDate; }
        }

        // Property with protected setter
        public bool NeedsFertilizer
        {
            get { return _needsFertilizer; }
            protected set { _needsFertilizer = value; }
        }

        // Calculated property
        public int DaysSinceAcquisition
        {
            get { return (DateTime.Now - _acquisitionDate).Days; }
        }

        // Constructor
        public Plant(string name, string species, string lightNeeds, string waterNeeds, decimal price)
        {
            Name = name; // Using the property for validation
            Species = species; // Using the property for validation
            LightNeeds = lightNeeds;
            WaterNeeds = waterNeeds;
            Price = price; // Using the property for validation
            _acquisitionDate = DateTime.Now;
            _needsFertilizer = true; // Most plants need fertilizer by default
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
            Console.WriteLine($"Needs Fertilizer: {(NeedsFertilizer ? "Yes" : "No")}");
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
            NeedsFertilizer = needsFertilizer; // Using the property
            Console.WriteLine($"Updated fertilizer needs for {Name}: {(NeedsFertilizer ? "Needs fertilizer" : "Does not need fertilizer")}");
        }
    }
}
```

In this updated class:
- We've replaced the public fields with private fields and public properties
- We've added validation to some properties to ensure data integrity
- We've created a read-only property (`AcquisitionDate`) that can only be set in the constructor or by methods within the class
- We've created a property with a protected setter (`NeedsFertilizer`) that can only be set by the class itself or derived classes
- We've added a calculated property (`DaysSinceAcquisition`) that computes its value based on other data

## Auto-Implemented Properties

C# provides a shorthand syntax for properties that don't require custom logic in their accessors. These are called auto-implemented properties.

Let's update our `FloweringPlant` class to use auto-implemented properties:

```csharp
// Models/FloweringPlant.cs
using System;

namespace ExtraVert.Models
{
    public class FloweringPlant : Plant
    {
        // Auto-implemented properties
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

            // Use the protected property from the base class
            if (isAnnual)
            {
                // Annual plants typically need more fertilizer
                NeedsFertilizer = true;
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
            string fertilizerInstructions = NeedsFertilizer
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
            NeedsFertilizer = true;
        }
    }
}
```

In this updated class:
- We've used auto-implemented properties for `FlowerColor`, `IsAnnual`, and `BloomingSeason`
- We've updated the code to use the `NeedsFertilizer` property instead of calling the `UpdateFertilizerNeeds` method

## Expression-Bodied Properties

C# 6.0 introduced expression-bodied members, which provide a more concise syntax for methods and properties that consist of a single expression.

Let's create a new class called `PlantStats` that uses expression-bodied properties:

```csharp
// Models/PlantStats.cs
using System;
using System.Collections.Generic;
using System.Linq;

namespace ExtraVert.Models
{
    public class PlantStats
    {
        private List<Plant> _plants;

        public PlantStats(List<Plant> plants)
        {
            _plants = plants;
        }

        // Expression-bodied properties
        public int TotalPlants => _plants.Count;

        public decimal TotalValue => _plants.Sum(p => p.Price);

        public decimal AveragePrice => TotalPlants > 0 ? TotalValue / TotalPlants : 0;

        public int FloweringPlantCount => _plants.Count(p => p is FloweringPlant);

        public int FoliagePlantCount => _plants.Count(p => p is FoliagePlant);

        public int SucculentPlantCount => _plants.Count(p => p is SucculentPlant);

        public Plant MostExpensivePlant => _plants.OrderByDescending(p => p.Price).FirstOrDefault();

        public Plant LeastExpensivePlant => _plants.OrderBy(p => p.Price).FirstOrDefault();

        // Method to display stats
        public void DisplayStats()
        {
            Console.WriteLine("Plant Inventory Statistics:");
            Console.WriteLine($"Total Plants: {TotalPlants}");
            Console.WriteLine($"Total Inventory Value: ${TotalValue}");
            Console.WriteLine($"Average Plant Price: ${AveragePrice:F2}");
            Console.WriteLine($"Flowering Plants: {FloweringPlantCount}");
            Console.WriteLine($"Foliage Plants: {FoliagePlantCount}");
            Console.WriteLine($"Succulent Plants: {SucculentPlantCount}");

            if (MostExpensivePlant != null)
            {
                Console.WriteLine($"Most Expensive Plant: {MostExpensivePlant.Name} (${MostExpensivePlant.Price})");
            }

            if (LeastExpensivePlant != null)
            {
                Console.WriteLine($"Least Expensive Plant: {LeastExpensivePlant.Name} (${LeastExpensivePlant.Price})");
            }
        }
    }
}
```

In this class:
- We've used expression-bodied properties for all properties
- Each property is defined with the `=>` syntax followed by a single expression
- The properties are calculated on-demand based on the current state of the `_plants` list

## Property Accessors and Access Modifiers

Property accessors (`get` and `set`) can have their own access modifiers, which allows for more fine-grained control over property access.

Let's create a new class called `Inventory` that demonstrates this:

```csharp
// Models/Inventory.cs
using System;
using System.Collections.Generic;

namespace ExtraVert.Models
{
    public class Inventory
    {
        private List<Plant> _plants = new List<Plant>();
        private decimal _totalValue;

        // Property with public getter and private setter
        public decimal TotalValue
        {
            get { return _totalValue; }
            private set { _totalValue = value; }
        }

        // Property with different access modifiers for getter and setter
        public int Count
        {
            get { return _plants.Count; }
        }

        // Method to add a plant to the inventory
        public void AddPlant(Plant plant)
        {
            _plants.Add(plant);
            TotalValue += plant.Price; // Update the total value
        }

        // Method to remove a plant from the inventory
        public bool RemovePlant(Plant plant)
        {
            bool removed = _plants.Remove(plant);
            if (removed)
            {
                TotalValue -= plant.Price; // Update the total value
            }
            return removed;
        }

        // Method to get all plants
        public List<Plant> GetAllPlants()
        {
            return new List<Plant>(_plants); // Return a copy to prevent direct modification
        }

        // Method to get plants by type
        public List<Plant> GetPlantsByType(string plantType)
        {
            return _plants.FindAll(p => p.GetPlantType().Equals(plantType, StringComparison.OrdinalIgnoreCase));
        }
    }
}
```

In this class:
- We've created a property `TotalValue` with a public getter and a private setter, which means it can be read from outside the class but only set from within the class
- We've created a property `Count` with only a getter, making it effectively read-only
- We've provided methods to add and remove plants from the inventory, which update the `TotalValue` property
- We've provided methods to get plants from the inventory, returning copies to prevent direct modification of the internal list

## Updating the Program

Let's update the `Program.cs` file to demonstrate the use of our enhanced classes:

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

            // Create a PlantStats object
            PlantStats stats = new PlantStats(plantRepo.GetAllPlants());

            // Create an Inventory object
            Inventory inventory = new Inventory();

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

                    case 5: // Exit
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

We've updated the program to use our new `Inventory` class and to display inventory statistics using the `PlantStats` class.

## Updating the Menu

Let's update the `Menu` class to include the new "View Inventory Stats" option:

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
            Console.WriteLine("5. Exit");
            Console.Write("Enter your choice (1-5): ");
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

## Running the Application

Now that we've updated our application with enhanced properties and encapsulation, let's run it to see the changes:

```bash
dotnet run
```

You should see the welcome message and the main menu. You can select option 1 to view all plants, which will now display additional information like the days since acquisition. You can also select option 4 to view inventory statistics.

## Property Guidelines

Here are some guidelines for using properties effectively:

1. **Use properties instead of public fields**: Properties provide better encapsulation and allow for validation, calculated values, and change notification.

2. **Keep property getters side-effect free**: Property getters should not change the state of the object or have other side effects.

3. **Make properties with minimal logic**: If a property requires complex logic, consider using a method instead.

4. **Use auto-implemented properties for simple cases**: When you don't need custom logic in the accessors, use auto-implemented properties for brevity.

5. **Consider making properties read-only when appropriate**: If a property should not be changed after initialization, make it read-only.

6. **Use expression-bodied properties for simple computed properties**: When a property is a simple expression, use the expression-bodied syntax for brevity.

7. **Validate input in property setters**: Use property setters to validate input and maintain invariants.

## Next Steps

In the next chapter, we'll explore methods and behaviors in C#. We'll learn how to define and call methods, how to pass parameters, and how to return values.

Before moving on, make sure you're comfortable with:
- Creating and using properties with getters and setters
- Implementing auto-implemented properties
- Creating read-only and write-only properties
- Implementing property validation
- Using expression-bodied properties
- Understanding property accessors and their access modifiers

## Practice Exercise

Enhance the ExtraVert Garden application by:
1. Adding a new property to the `Plant` class called `LastWateredDate` with a private setter
2. Adding a method to the `Plant` class called `WaterPlant` that updates the `LastWateredDate` property
3. Adding a calculated property to the `Plant` class called `DaysSinceLastWatered` that returns the number of days since the plant was last watered
4. Updating the `DisplayInfo` method to show the last watered date and days since last watered
5. Updating the `Water` method to call the `WaterPlant` method
6. Running the application and verifying that the watering information is displayed correctly