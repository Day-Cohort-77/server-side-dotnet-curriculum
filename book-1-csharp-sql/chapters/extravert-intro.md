# ExtraVert Garden: Project Setup

Welcome to the ExtraVert Garden project! In this series of chapters, you'll build a console application for managing a plant nursery. Through this project, you'll learn about object-oriented programming concepts in C#, including classes, inheritance, properties, methods, and interfaces.

## Project Overview

ExtraVert is a plant nursery that needs a management system to keep track of their inventory. The application will allow users to:

- View all plants in the inventory
- Add new plants to the inventory
- Search for plants by name or type
- View details about specific plants
- Track plant care information
- Calculate statistics about the inventory

Throughout this project, you'll build this application step by step, learning important object-oriented programming concepts along the way.

## Setting Up the Project

Let's start by creating a new console application for the ExtraVert Garden project:

```bash
dotnet new console -n ExtraVert
cd ExtraVert
code .
```

This creates a new console application named "ExtraVert" and opens it in Visual Studio Code.

## Project Structure

Let's plan the structure of our application. We'll organize our code into several files and directories:

- `Program.cs`: The main entry point of the application
- `Models/`: A directory for our model classes
  - `Plant.cs`: The base class for all plants
  - `FloweringPlant.cs`: A derived class for flowering plants
  - `FoliagePlant.cs`: A derived class for foliage plants
  - `SucculentPlant.cs`: A derived class for succulent plants
- `Data/`: A directory for data management
  - `PlantRepository.cs`: A class for managing plant data
- `UI/`: A directory for user interface code
  - `Menu.cs`: A class for handling the menu system

Let's create these directories:

```bash
mkdir Models
mkdir Data
mkdir UI
```

## Creating the Initial Files

Now, let's create the initial files for our project. First, let's modify the `Program.cs` file:

```csharp
// Program.cs
using System;

namespace ExtraVert
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Welcome to ExtraVert Plant Nursery Management System!");
            Console.WriteLine("------------------------------------------------");
            Console.WriteLine();

            // We'll add more code here in future chapters

            Console.WriteLine("Press any key to exit...");
            Console.ReadKey();
        }
    }
}
```

Next, let's create a basic `Plant` class in the Models directory:

```csharp
// Models/Plant.cs
using System;

namespace ExtraVert.Models
{
    // Define the custom type
    public class Plant
    {
        // Define the properties that will be on each object of this type
        public string Name { get; set; }
        public string Species { get; set; }
        public string LightNeeds { get; set; }
        public string WaterNeeds { get; set; }
        public decimal Price { get; set; }

        // The constructor (i.e. what happens when a new object is created)
        public Plant(string name, string species, string lightNeeds, string waterNeeds, decimal price)
        {
            Name = name;
            Species = species;
            LightNeeds = lightNeeds;
            WaterNeeds = waterNeeds;
            Price = price;
        }

        // A method that is available on every generated object
        public void DisplayInfo()
        {
            Console.WriteLine($"Name: {Name}");
            Console.WriteLine($"Species: {Species}");
            Console.WriteLine($"Light Needs: {LightNeeds}");
            Console.WriteLine($"Water Needs: {WaterNeeds}");
            Console.WriteLine($"Price: ${Price}");
        }
    }
}
```

Let's also create a simple `PlantRepository` class to manage our plant data:

```csharp
// Data/PlantRepository.cs
using System;
using System.Collections.Generic;
using ExtraVert.Models;

namespace ExtraVert.Data
{
    public class PlantRepository
    {
        // A collection of plants
        private List<Plant> _plants = new List<Plant>();

        // A method that returns the list of plants
        public List<Plant> GetAllPlants()
        {
            return _plants;
        }

        // A method to add a plant to the private list
        public void AddPlant(Plant plant)
        {
            _plants.Add(plant);
        }

        // A method to insert some starter plants into the list
        public void SeedData()
        {
            // Add some sample plants
            _plants.Add(new Plant("Snake Plant", "Sansevieria trifasciata", "Low to bright indirect", "Low", 15.99m));
            _plants.Add(new Plant("Monstera", "Monstera deliciosa", "Bright indirect", "Moderate", 29.99m));
            _plants.Add(new Plant("Peace Lily", "Spathiphyllum", "Low to bright indirect", "Moderate", 19.99m));
        }
    }
}
```

Finally, let's create a basic `Menu` class to handle the user interface:

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
            Console.WriteLine("4. Exit");
            Console.Write("Enter your choice (1-4): ");
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

## Updating Program.cs to Use Our Classes

Now, let's update the `Program.cs` file to use our newly created classes:

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
            Console.WriteLine("Welcome to ExtraVert Plant Nursery Management System!");
            Console.WriteLine("------------------------------------------------");
            Console.WriteLine();

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
                        Console.WriteLine("This feature will be implemented in a future chapter.");
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

## Running the Application

Now that we've set up our project structure and created the initial files, let's run the application to see it in action:

```bash
dotnet run
```

You should see the welcome message and the main menu. You can select option 1 to view all plants, which will display the three sample plants we added in the `SeedData` method.

## Project Planning

As we continue to build the ExtraVert Garden application, we'll implement the following features:

1. **Plant Classes**: We'll create a hierarchy of plant classes to represent different types of plants.
2. **Plant Management**: We'll implement functionality to add, view, and search for plants.
3. **Plant Properties**: We'll add more properties to our plant classes to track additional information.
4. **Plant Methods**: We'll implement methods to perform actions on plants, such as watering or fertilizing.
5. **Interfaces**: We'll use interfaces to define common behavior for different types of plants.

## Next Steps

In the next chapter, we'll expand our application by creating derived plant classes to represent different types of plants. We'll learn about inheritance and how to create a class hierarchy.

Before moving on, make sure you're comfortable with:
- Setting up a console application project
- Creating classes and organizing them into directories
- Understanding the basic structure of the ExtraVert Garden application
- Running the application and navigating the menu

## Practice Exercise

Modify the ExtraVert Garden application to:
1. Add two more sample plants to the `SeedData` method
2. Enhance the `Plant` class with an additional property (e.g., `PlantedDate` or `Size`)
3. Update the `DisplayInfo` method to show the new property
4. Run the application and verify that the new plants and property are displayed correctly

[Next Chapter: Plant Classes](./extravert-plant-classes.md)