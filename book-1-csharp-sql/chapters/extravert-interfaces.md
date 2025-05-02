# Interfaces and Abstraction

In this chapter, we'll explore interfaces in C#, which are a powerful tool for achieving abstraction and creating flexible, maintainable code. Interfaces define a contract that classes can implement, allowing for polymorphism and loose coupling.

## Learning Objectives

By the end of this chapter, you should be able to:
- Understand the concept of interfaces in C#
- Define and implement interfaces
- Use interfaces to achieve polymorphism
- Implement multiple interfaces in a single class
- Understand default interface methods (C# 8.0+)
- Apply the Interface Segregation Principle
- Use interfaces for dependency injection

## Understanding Interfaces

An interface is a contract that defines a set of methods, properties, events, or indexers that a class must implement. Interfaces allow you to define what a class can do without specifying how it does it.

Key characteristics of interfaces:
- They can contain method signatures, properties, events, and indexers, but not fields
- They cannot contain implementation code (except for default interface methods in C# 8.0+)
- A class can implement multiple interfaces
- Interfaces can inherit from other interfaces

## Creating an Interface

Let's create an interface for plants that can be propagated:

```csharp
// Models/IPropagatable.cs
using System;

namespace ExtraVert.Models
{
    public interface IPropagatable
    {
        // Method signatures
        void Propagate();
        bool CanPropagate();

        // Property signatures
        string PropagationMethod { get; }
        int PropagationSuccessRate { get; }

        // Default method (C# 8.0+)
        string GetPropagationInstructions() => $"Propagate using {PropagationMethod} method. Success rate: {PropagationSuccessRate}%.";
    }
}
```
## Implementing an Interface

Now, let's update our plant classes to implement the `IPropagatable` interface. First, let's update the `SucculentPlant` class:

```csharp
// Models/SucculentPlant.cs
using System;

namespace ExtraVert.Models
{
    public class SucculentPlant : Plant, IPropagatable
    {
        // Additional properties specific to succulent plants
        public bool IsIndoor { get; set; }
        public string WaterStorageOrgan { get; set; }
        public bool IsToxicToPets { get; set; }

        // Properties from IPropagatable interface
        public string PropagationMethod { get; private set; }
        public int PropagationSuccessRate { get; private set; }

        // Constructor
        public SucculentPlant(string name, string species, string lightNeeds, string waterNeeds,
                             decimal price, bool isIndoor, string waterStorageOrgan, bool isToxicToPets)
            : base(name, species, lightNeeds, waterNeeds, price)
        {
            IsIndoor = isIndoor;
            WaterStorageOrgan = waterStorageOrgan;
            IsToxicToPets = isToxicToPets;

            // Initialize propagation properties
            PropagationMethod = "Leaf or stem cutting";
            PropagationSuccessRate = 90; // Succulents are generally easy to propagate
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

            // Add propagation information if applicable
            if (this is IPropagatable propagatable)
            {
                Console.WriteLine($"Propagation Method: {propagatable.PropagationMethod}");
                Console.WriteLine($"Propagation Success Rate: {propagatable.PropagationSuccessRate}%");
            }
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
            LastWateredDate = DateTime.Now;
        }

        // Implement IPropagatable.Propagate method
        public void Propagate()
        {
            Console.WriteLine($"Propagating {Name} by taking a leaf or stem cutting.");
            Console.WriteLine("1. Cut a healthy leaf or stem segment.");
            Console.WriteLine("2. Allow the cutting to callus over for a few days.");
            Console.WriteLine("3. Place the cutting on well-draining soil.");
            Console.WriteLine("4. Water sparingly until roots develop.");
        }

        // Implement IPropagatable.CanPropagate method
        public bool CanPropagate()
        {
            // Most succulents can be propagated year-round, but let's add some logic
            // For demonstration, let's say succulents can be propagated if they're at least 30 days old
            return DaysSinceAcquisition >= 30;
        }
    }
}
```

Now, let's update the `FloweringPlant` class to implement the `IPropagatable` interface:

```csharp
// Models/FloweringPlant.cs
using System;

namespace ExtraVert.Models
{
    public class FloweringPlant : Plant, IPropagatable
    {
        // Additional properties specific to flowering plants
        public string FlowerColor { get; set; }
        public bool IsAnnual { get; set; }
        public string BloomingSeason { get; set; }

        // Properties from IPropagatable interface
        public string PropagationMethod { get; private set; }
        public int PropagationSuccessRate { get; private set; }

        // Constructor
        public FloweringPlant(string name, string species, string lightNeeds, string waterNeeds,
                             decimal price, string flowerColor, bool isAnnual, string bloomingSeason)
            : base(name, species, lightNeeds, waterNeeds, price)
        {
            FlowerColor = flowerColor;
            IsAnnual = isAnnual;
            BloomingSeason = bloomingSeason;

            // Initialize propagation properties
            PropagationMethod = IsAnnual ? "Seeds" : "Division or cuttings";
            PropagationSuccessRate = IsAnnual ? 80 : 70; // Annual plants are often propagated by seeds

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

            // Add propagation information if applicable
            if (this is IPropagatable propagatable)
            {
                Console.WriteLine($"Propagation Method: {propagatable.PropagationMethod}");
                Console.WriteLine($"Propagation Success Rate: {propagatable.PropagationSuccessRate}%");
            }
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

        // Implement IPropagatable.Propagate method
        public void Propagate()
        {
            if (IsAnnual)
            {
                Console.WriteLine($"Propagating {Name} by collecting and sowing seeds.");
                Console.WriteLine("1. Allow flowers to go to seed.");
                Console.WriteLine("2. Collect seeds when they are mature.");
                Console.WriteLine("3. Store seeds in a cool, dry place.");
                Console.WriteLine("4. Sow seeds in appropriate growing medium during the next growing season.");
            }
            else
            {
                Console.WriteLine($"Propagating {Name} by division or cuttings.");
                Console.WriteLine("1. Take a cutting from a healthy stem or divide the root ball.");
                Console.WriteLine("2. For cuttings, dip in rooting hormone and plant in growing medium.");
                Console.WriteLine("3. For divisions, replant each division in its own pot.");
                Console.WriteLine("4. Keep soil moist until new growth appears.");
## Creating a Propagation Service

Now, let's create a service class that works with propagatable plants:

```csharp
// Services/PropagationService.cs
using System;
using System.Collections.Generic;
using System.Linq;
using ExtraVert.Models;

namespace ExtraVert.Services
{
    public class PropagationService
    {
        // Method that works with any IPropagatable object
        public void PropagateIfPossible(IPropagatable plant)
        {
            if (plant.CanPropagate())
            {
                Console.WriteLine($"Propagating plant using {plant.PropagationMethod} method...");
                plant.Propagate();
                Console.WriteLine($"Expected success rate: {plant.PropagationSuccessRate}%");
            }
            else
            {
                Console.WriteLine("This plant cannot be propagated at this time.");
                Console.WriteLine(plant.GetPropagationInstructions());
            }
        }

        // Method that filters a list of plants to find those that can be propagated
        public List<IPropagatable> GetPropagatablePlants(List<Plant> plants)
        {
            return plants
                .OfType<IPropagatable>() // Filter to only IPropagatable plants
                .Where(p => p.CanPropagate()) // Filter to only those that can be propagated now
                .ToList();
        }

        // Method that provides propagation instructions for all propagatable plants
        public void DisplayPropagationInstructions(List<Plant> plants)
        {
            var propagatablePlants = plants.OfType<IPropagatable>().ToList();

            if (propagatablePlants.Count == 0)
            {
                Console.WriteLine("No propagatable plants found.");
                return;
            }

            Console.WriteLine("Propagation Instructions:");
            Console.WriteLine("------------------------");

            foreach (var plant in propagatablePlants)
            {
                // We need to cast to access the Name property
                var actualPlant = plant as Plant;
                Console.WriteLine($"{actualPlant?.Name ?? "Unknown Plant"}:");
                Console.WriteLine($"Method: {plant.PropagationMethod}");
                Console.WriteLine($"Success Rate: {plant.PropagationSuccessRate}%");
                Console.WriteLine($"Can Propagate Now: {(plant.CanPropagate() ? "Yes" : "No")}");
                Console.WriteLine(plant.GetPropagationInstructions());
                Console.WriteLine();
            }
        }
    }
}
```

In this service class:
- We've created methods that work with `IPropagatable` objects, regardless of their concrete type
- We've used LINQ's `OfType<T>()` method to filter a list of plants to only those that implement `IPropagatable`
- We've demonstrated how interfaces enable polymorphism by allowing different types of plants to be treated uniformly as `IPropagatable` objects

## Creating Multiple Interfaces

Let's create another interface for plants that can bloom:

```csharp
// Models/IBloomable.cs
using System;

namespace ExtraVert.Models
{
    public interface IBloomable
    {
        // Method signatures
        void Bloom();
        bool IsInBloomingSeason();

        // Property signatures
        string BloomColor { get; }
        string BloomingSeason { get; }

        // Default method (C# 8.0+)
        string GetBloomingInstructions() => $"This plant blooms in {BloomingSeason} with {BloomColor} flowers.";
    }
}
```

Now, let's update the `FloweringPlant` class to implement both `IPropagatable` and `IBloomable`:

```csharp
// Models/FloweringPlant.cs
using System;

namespace ExtraVert.Models
{
    public class FloweringPlant : Plant, IPropagatable, IBloomable
    {
        // Additional properties specific to flowering plants
        public string FlowerColor { get; set; }
        public bool IsAnnual { get; set; }
        public string BloomingSeason { get; set; }

        // Properties from IPropagatable interface
        public string PropagationMethod { get; private set; }
        public int PropagationSuccessRate { get; private set; }

        // Properties from IBloomable interface
        public string BloomColor => FlowerColor; // Implement IBloomable.BloomColor using existing property
        string IBloomable.BloomingSeason => BloomingSeason; // Explicit interface implementation

        // Constructor
        public FloweringPlant(string name, string species, string lightNeeds, string waterNeeds,
                             decimal price, string flowerColor, bool isAnnual, string bloomingSeason)
            : base(name, species, lightNeeds, waterNeeds, price)
        {
            FlowerColor = flowerColor;
            IsAnnual = isAnnual;
            BloomingSeason = bloomingSeason;

            // Initialize propagation properties
            PropagationMethod = IsAnnual ? "Seeds" : "Division or cuttings";
            PropagationSuccessRate = IsAnnual ? 80 : 70; // Annual plants are often propagated by seeds

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

            // Add propagation information if applicable
            if (this is IPropagatable propagatable)
            {
                Console.WriteLine($"Propagation Method: {propagatable.PropagationMethod}");
                Console.WriteLine($"Propagation Success Rate: {propagatable.PropagationSuccessRate}%");
            }

            // Add blooming information if applicable
            if (this is IBloomable bloomable)
            {
                Console.WriteLine($"Blooming Season: {bloomable.BloomingSeason}");
                Console.WriteLine($"Bloom Color: {bloomable.BloomColor}");
                Console.WriteLine($"Currently Blooming: {(bloomable.IsInBloomingSeason() ? "Yes" : "No")}");
            }
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

        // Implement IPropagatable.Propagate method
        public void Propagate()
        {
            if (IsAnnual)
            {
                Console.WriteLine($"Propagating {Name} by collecting and sowing seeds.");
                Console.WriteLine("1. Allow flowers to go to seed.");
                Console.WriteLine("2. Collect seeds when they are mature.");
                Console.WriteLine("3. Store seeds in a cool, dry place.");
                Console.WriteLine("4. Sow seeds in appropriate growing medium during the next growing season.");
            }
            else
            {
                Console.WriteLine($"Propagating {Name} by division or cuttings.");
                Console.WriteLine("1. Take a cutting from a healthy stem or divide the root ball.");
                Console.WriteLine("2. For cuttings, dip in rooting hormone and plant in growing medium.");
                Console.WriteLine("3. For divisions, replant each division in its own pot.");
                Console.WriteLine("4. Keep soil moist until new growth appears.");
            }
        }

        // Implement IPropagatable.CanPropagate method
        public bool CanPropagate()
        {
            if (IsAnnual)
            {
                // Annual plants can be propagated by seed when they're flowering
                return BloomingSeason.Contains(DateTime.Now.ToString("MMMM"), StringComparison.OrdinalIgnoreCase);
            }
            else
            {
                // Perennial plants can be propagated during their active growing season
                return !BloomingSeason.Contains("Winter", StringComparison.OrdinalIgnoreCase);
            }
        }

        // Implement IBloomable.Bloom method
        public void Bloom()
        {
            Console.WriteLine($"{Name} is blooming with beautiful {BloomColor} flowers!");
        }

        // Implement IBloomable.IsInBloomingSeason method
        public bool IsInBloomingSeason()
        {
            // Check if the current month is in the blooming season
            return BloomingSeason.Contains(DateTime.Now.ToString("MMMM"), StringComparison.OrdinalIgnoreCase);
        }
    }
}
```
## Creating a Blooming Service

Now, let's create a service class that works with bloomable plants:

```csharp
// Services/BloomingService.cs
using System;
using System.Collections.Generic;
using System.Linq;
using ExtraVert.Models;

namespace ExtraVert.Services
{
    public class BloomingService
    {
        // Method that works with any IBloomable object
        public void TriggerBloom(IBloomable plant)
        {
            if (plant.IsInBloomingSeason())
            {
                Console.WriteLine("Triggering bloom...");
                plant.Bloom();
            }
            else
            {
                Console.WriteLine("This plant is not in its blooming season.");
                Console.WriteLine(plant.GetBloomingInstructions());
            }
        }

        // Method that filters a list of plants to find those that are currently blooming
        public List<IBloomable> GetCurrentlyBloomingPlants(List<Plant> plants)
        {
            return plants
                .OfType<IBloomable>() // Filter to only IBloomable plants
                .Where(p => p.IsInBloomingSeason()) // Filter to only those that are in blooming season
                .ToList();
        }

        // Method that provides blooming information for all bloomable plants
        public void DisplayBloomingInformation(List<Plant> plants)
        {
            var bloomablePlants = plants.OfType<IBloomable>().ToList();

            if (bloomablePlants.Count == 0)
            {
                Console.WriteLine("No bloomable plants found.");
                return;
            }

            Console.WriteLine("Blooming Information:");
            Console.WriteLine("--------------------");

            foreach (var plant in bloomablePlants)
            {
                // We need to cast to access the Name property
                var actualPlant = plant as Plant;
                Console.WriteLine($"{actualPlant?.Name ?? "Unknown Plant"}:");
                Console.WriteLine($"Bloom Color: {plant.BloomColor}");
                Console.WriteLine($"Blooming Season: {plant.BloomingSeason}");
                Console.WriteLine($"Currently Blooming: {(plant.IsInBloomingSeason() ? "Yes" : "No")}");
                Console.WriteLine(plant.GetBloomingInstructions());
                Console.WriteLine();
            }
        }
    }
}
```

## Interface Segregation Principle

The Interface Segregation Principle (ISP) is one of the SOLID principles of object-oriented design. It states that "Clients should not be forced to depend upon interfaces that they do not use."

In our example, we've applied the ISP by creating separate interfaces for different behaviors:
- `IPropagatable` for plants that can be propagated
- `IBloomable` for plants that can bloom

This allows classes to implement only the interfaces that are relevant to them. For example, a `SucculentPlant` might implement `IPropagatable` but not `IBloomable`, while a `FloweringPlant` might implement both.

## Dependency Injection with Interfaces

Interfaces are commonly used for dependency injection, a technique where dependencies are provided to a class rather than created by the class itself. This promotes loose coupling and makes code more testable.

Let's create a `PlantManager` class that uses dependency injection:

```csharp
// Services/PlantManager.cs
using System;
using System.Collections.Generic;
using ExtraVert.Models;

namespace ExtraVert.Services
{
    // Interface for plant repositories
    public interface IPlantRepository
    {
        List<Plant> GetAllPlants();
        void AddPlant(Plant plant);
        List<Plant> SearchPlants(string searchTerm);
    }

    // PlantManager class that depends on IPlantRepository
    public class PlantManager
    {
        private readonly IPlantRepository _plantRepository;

        // Constructor injection
        public PlantManager(IPlantRepository plantRepository)
        {
            _plantRepository = plantRepository;
        }

        // Methods that use the injected repository
        public List<Plant> GetAllPlants()
        {
            return _plantRepository.GetAllPlants();
        }

        public void AddPlant(Plant plant)
        {
            _plantRepository.AddPlant(plant);
        }

        public List<Plant> SearchPlants(string searchTerm)
        {
            return _plantRepository.SearchPlants(searchTerm);
        }

        // Additional methods that use the repository
        public List<Plant> GetPlantsByType(string plantType)
        {
            return _plantRepository.GetAllPlants().FindAll(p =>
                p.GetPlantType().Equals(plantType, StringComparison.OrdinalIgnoreCase));
        }

        public decimal GetTotalInventoryValue()
        {
            decimal total = 0;
            foreach (var plant in _plantRepository.GetAllPlants())
            {
                total += plant.Price;
            }
            return total;
        }
    }
}
```

Now, let's update our `PlantRepository` class to implement the `IPlantRepository` interface:

```csharp
// Data/PlantRepository.cs
using System;
using System.Collections.Generic;
using ExtraVert.Models;
using ExtraVert.Services;

namespace ExtraVert.Data
{
    public class PlantRepository : IPlantRepository
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
            // Add sample plants (implementation omitted for brevity)
        }
    }
}
```

## Interface Guidelines

Here are some guidelines for using interfaces effectively:

1. **Keep interfaces focused**: Each interface should have a clear, specific purpose. Follow the Interface Segregation Principle.

2. **Name interfaces appropriately**: Interface names typically start with "I" followed by a descriptive name (e.g., `IPropagatable`, `IBloomable`).

3. **Design for extensibility**: Interfaces should be designed to accommodate future changes without breaking existing code.

4. **Use interfaces for abstraction**: Interfaces help abstract away implementation details, allowing you to focus on what an object does rather than how it does it.

5. **Use interfaces for polymorphism**: Interfaces enable polymorphism, allowing different types to be treated uniformly.

6. **Use interfaces for dependency injection**: Interfaces make it easier to swap out implementations, which is useful for testing and maintaining code.

7. **Consider default interface methods**: In C# 8.0+, you can provide default implementations for interface methods, which can be useful for adding new methods to existing interfaces without breaking compatibility.

## Next Steps

In this chapter, we've explored interfaces and how they can be used to create more flexible, maintainable code. We've implemented interfaces for our plant classes, created service classes that work with interfaces, and learned about the Interface Segregation Principle and dependency injection.

In the next chapters, we'll explore SQL and database connectivity, which will allow us to persist our plant data beyond the lifetime of our application.

Before moving on, make sure you're comfortable with:
- Defining and implementing interfaces
- Using interfaces to achieve polymorphism
- Implementing multiple interfaces in a single class
- Understanding the Interface Segregation Principle
- Using interfaces for dependency injection

## Practice Exercise

Enhance the ExtraVert Garden application by:
1. Creating a new interface called `ISeasonal` with methods like `IsInSeason()` and properties like `ActiveSeason`
2. Implementing the `ISeasonal` interface in appropriate plant classes
3. Creating a `SeasonalService` class that works with `ISeasonal` objects
4. Updating the program to use the new interface and service
5. Running the application and verifying that the seasonal functionality works correctly
            }
        }

        // Implement IPropagatable.CanPropagate method
        public bool CanPropagate()
        {
            if (IsAnnual)
            {
                // Annual plants can be propagated by seed when they're flowering
                return BloomingSeason.Contains(DateTime.Now.ToString("MMMM"), StringComparison.OrdinalIgnoreCase);
            }
            else
            {
                // Perennial plants can be propagated during their active growing season
                return !BloomingSeason.Contains("Winter", StringComparison.OrdinalIgnoreCase);
            }
        }
    }
}
```

In this interface:
- We've defined two method signatures: `Propagate` and `CanPropagate`
- We've defined two property signatures: `PropagationMethod` and `PropagationSuccessRate`
- We've included a default implementation for the `GetPropagationInstructions` method (C# 8.0+)