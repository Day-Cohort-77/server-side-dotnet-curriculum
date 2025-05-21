# Setting Up a Test Project

## Learning Objectives
- Create a new repository from the TestTube template
- Understand the structure of the TestTube solution
- Set up a test project with the necessary packages
- Configure the test project to work with the API

## Creating Your Repository from the Template

The first step is to create your own repository from the TestTube template. This will give you a copy of the project that you can modify and work with.

1. Navigate to the TestTube template repository at https://github.com/nashville-software-school/TestTube
2. Click the green "Use this template" button at the top of the page
3. Select "Create a new repository"
4. Name your repository "TestTube" (or another name of your choice)
5. Choose whether to make your repository public or private
6. Click "Create repository from template"

Once your repository is created, you can clone it to your local machine:

```bash
git clone https://github.com/YOUR-USERNAME/TestTube.git
cd TestTube
```

Replace `YOUR-USERNAME` with your GitHub username.

## Understanding the Solution Structure

The TestTube solution contains two projects:

1. **TestTubes.API**: The main API project with controllers, models, and database context
2. **TestTubes.Tests**: The test project where we'll write our integration tests

Let's take a closer look at the API project structure:

```
TestTubes.API/
├── Controllers/
│   ├── EquipmentController.cs
│   └── ScientistController.cs
├── Data/
│   └── TestTubeDbContext.cs
├── Models/
│   ├── Equipment.cs
│   └── Scientist.cs
├── Program.cs
└── appsettings.json
```

The API project is a standard ASP.NET Core API with Entity Framework Core. It includes:

- Controllers for managing equipment and scientists
- Models representing the domain entities
- A DbContext for database operations
- Program.cs that configures and runs the application

## Setting Up the Test Project

The test project is already set up in the template, but let's understand what's included:

```
TestTubes.Tests/
├── EquipmentTests.cs
├── ScientistTests.cs
└── TestHelper.cs
```

The test project includes:

- Test classes for each resource
- A TestHelper class that we'll explore in detail later

## Required NuGet Packages

The test project already includes the necessary NuGet packages, but it's important to understand what they are and why we need them:

1. **Microsoft.AspNetCore.Mvc.Testing**: Provides the WebApplicationFactory for hosting the API during tests
2. **Microsoft.EntityFrameworkCore.InMemory**: Allows us to use an in-memory database for testing
3. **xUnit**: The testing framework we'll use
4. **xUnit.runner.visualstudio**: Enables running tests in Visual Studio
5. **coverlet.collector**: Collects code coverage information

If you need to add these packages manually in a different project, you can use the .NET CLI:

```bash
dotnet add package Microsoft.AspNetCore.Mvc.Testing --version 8.0.0
dotnet add package Microsoft.EntityFrameworkCore.InMemory --version 8.0.0
dotnet add package xunit --version 2.6.1
dotnet add package xunit.runner.visualstudio --version 2.5.3
dotnet add package coverlet.collector --version 6.0.0
```

## Opening the Solution in Visual Studio Code

Now that we have our repository set up, let's open it in Visual Studio Code:

```bash
code .
```

Take some time to explore the solution structure and familiarize yourself with the code.

## Building the Solution

Before we start writing tests, let's make sure the solution builds correctly:

```bash
dotnet build
```

If everything is set up correctly, you should see a successful build message.

## Running the API

Let's also make sure the API runs correctly:

```bash
cd TestTubes.API
dotnet run
```

You should see output indicating that the API is running at the URL of `http://localhost:5081`

You can use Yaak to perform a GET request to `http://localhost:5081/equipment` to see a list of equipment items from the database.

Press `Ctrl+C` to stop the API when you're done.

## Understanding the Test Project Structure

Now let's take a closer look at the test project. Open the `TestTubes.Tests` directory and examine the files:

1. **TestHelper.cs**: Contains helper methods for setting up the test environment
2. **EquipmentTests.cs**: Contains tests for the equipment endpoints
3. **ScientistTests.cs**: Contains tests for the scientist endpoints

In the next chapters, we'll explore these files in detail and learn how to write effective integration tests.

## Next Steps

Now that we have our test project set up, we'll dive into the WebApplicationFactory, which is a key component for integration testing in ASP.NET Core.

[Understanding WebApplicationFactory](./testtube-webapplicationfactory.md)