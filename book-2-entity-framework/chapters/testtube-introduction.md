# Introduction to Integration Testing

## Learning Objectives
- Understand what integration testing is and why it's important
- Learn the difference between unit tests and integration tests
- Understand how integration tests fit into the testing pyramid
- Recognize when to use integration tests in your projects

## What is Integration Testing?

When developing software applications, testing is a critical part of ensuring your code works as expected. There are several types of tests you can write, each serving a different purpose. One important type is **integration testing**.

Integration testing verifies that different components of your application work correctly together. Unlike unit tests, which test individual components in isolation, integration tests examine how multiple components interact with each other.

In the context of an ASP.NET Core API with Entity Framework, integration tests typically involve:

1. Starting up a test version of your API
2. Making HTTP requests to your endpoints
3. Verifying that the responses are correct
4. Checking that data is properly stored in or retrieved from the database

## The Testing Pyramid

The testing pyramid is a concept that helps visualize the different types of tests and their relative quantities in a well-tested application:

```
         /\
        /  \
       /    \
      /      \
     /  E2E   \
    /----------\
   / Integration\
  /--------------\
 /      Unit      \
/__________________\
```

- **Unit Tests**: Form the base of the pyramid. They are numerous, fast, and test individual components in isolation.
- **Integration Tests**: The middle layer. They test how components work together.
- **End-to-End (E2E) Tests**: The top of the pyramid. They test the entire application from the user's perspective.

A well-balanced testing strategy includes all three types, with more unit tests than integration tests, and more integration tests than E2E tests.

## Why Integration Testing Matters

Integration tests are valuable because they:

1. **Catch issues that unit tests miss**: Unit tests can't catch problems that occur when components interact.
2. **Provide confidence in your API**: They verify that your endpoints work as expected from the client's perspective.
3. **Test database interactions**: They ensure your Entity Framework code correctly interacts with the database.
4. **Serve as documentation**: They demonstrate how your API is supposed to work.

## Integration Testing in ASP.NET Core

ASP.NET Core provides excellent tools for integration testing, including:

- **WebApplicationFactory**: Creates a test host for your API
- **TestServer**: Handles HTTP requests without requiring a real HTTP server
- **In-memory database provider**: Allows testing database operations without a real database

In this series of chapters, you'll learn how to use these tools to write effective integration tests for an ASP.NET Core API with Entity Framework.

## The TestTube Project

Throughout these chapters, we'll be working with the **TestTube** project, a simple API for managing laboratory equipment and scientists. The API is already fully functional, and our job will be to write integration tests to verify its behavior.

The TestTube API includes:

- Equipment management (CRUD operations)
- Scientist management (CRUD operations)
- Assignment of equipment to scientists

By the end of this series, you'll have implemented integration tests for key endpoints and gained a solid understanding of integration testing principles.

## Getting Started

1. Visit the [TestTube GitHub template repository](https://github.com/nashville-software-school/TestTube) template at  as our starting point.
2. Click the **Use this template** button
3. Choose **Create a new repository**
4. Choose your account and give it a name of **TestTube**
5. Once generated, clone the repo to your machine

## Next Steps

In the next chapter, there are instsructions on how you can set up an API+testing project in the future.

[Setting Up a Test Project](./testtube-setup.md)