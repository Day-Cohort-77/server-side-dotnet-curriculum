# Setting Up a Test Project from Scratch

## Learning Objectives
- Create a new ASP.NET Core Web API project
- Set up a test project with the necessary packages
- Configure the test project to work with the API
- Understand the structure of an integration test solution

## Creating Your API Project

Let's start by creating a new ASP.NET Core Web API project that we'll use for our integration tests.

1. Open a terminal and navigate to the directory where you want to create your project
2. Create a new solution:

```bash
dotnet new sln -n TestTube
```

3. Create a new ASP.NET Core Web API project:

```bash
dotnet new webapi -n TestTube.API
```

4. Add the API project to the solution:

```bash
dotnet sln add TestTube.API
```

## Setting Up the API Project

First, you would build your API project just as you have with the previous projects in this book. Once you have the basic API functionality working, you would then start building tests.

## Creating the Test Project

Now that we have our API project set up, let's create a test project to test it:

1. Navigate back to the solution directory:

```bash
cd ..
```

2. Create a new xUnit test project:

```bash
dotnet new xunit -n TestTube.Tests
```

3. Add the test project to the solution:

```bash
dotnet sln add TestTube.Tests
```

4. Add a reference to the API project from the test project:

```bash
cd TestTube.Tests
dotnet add reference ../TestTube.API
```

5. Add the necessary packages for integration testing:

```bash
dotnet add package Microsoft.AspNetCore.Mvc.Testing --version 8.0.0
dotnet add package Microsoft.EntityFrameworkCore.InMemory --version 8.0.0
dotnet add package FluentAssertions --version 6.12.0
dotnet add package Testcontainers.PostgreSql --version 3.9.0
```



You should see output indicating that all tests have passed.

## Understanding the Solution Structure

Let's take a moment to understand the structure of our solution:

```
TestTube/
├── TestTube.API/
│   ├── Controllers/
│   │   ├── EquipmentController.cs
│   │   └── ScientistController.cs
│   ├── Data/
│   │   └── TestTubeDbContext.cs
│   ├── Models/
│   │   ├── Equipment.cs
│   │   └── Scientist.cs
│   ├── Program.cs
│   └── appsettings.json
├── TestTube.Tests/
│   ├── EquipmentTests.cs
│   ├── ScientistTests.cs
│   └── TestHelper.cs
│   └── TestWebApplicationFactory.cs
└── TestTube.sln
```

This structure follows a common pattern for ASP.NET Core projects with integration tests:

1. **API Project**: Contains the controllers, models, and database context
2. **Test Project**: Contains the test classes and helper classes

The key components for integration testing are:

1. **WebApplicationFactory**: Creates a test host for your API
2. **TestHelper**: Encapsulates the complexity of setting up the test environment
3. **Test Classes**: Contain the actual test methods

## Next Steps

In the next few chapters, you will be learning about the concepts and reponsibilities of the modules in the  **TestTubes.Tests** project.

In the next we'll dive deeper into the WebApplicationFactory and how it works.

[Understanding WebApplicationFactory](./testtube-webapplicationfactory.md)