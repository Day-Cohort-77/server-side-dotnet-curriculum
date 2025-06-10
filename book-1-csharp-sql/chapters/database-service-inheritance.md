# Refactoring with Inheritance: Specialized Database Services

## Learning Objectives

By the end of this chapter, you should be able to:
- Identify when a class is becoming too large and needs refactoring
- Understand how inheritance can be used to create specialized database service classes
- Implement a base database service class with common functionality
- Create derived classes for specific resource types
- Update your application to use the new inheritance structure

## The Problem: Monolithic Database Service

As you've been working through the Jewelry Junction project, you've likely noticed that your `DatabaseService.cs` file has grown quite large. This is a common issue in software development known as the "God Object" anti-pattern, where a single class takes on too many responsibilities.

Let's examine the current state of our `DatabaseService.cs`:

1. It handles connections to the database
2. It contains methods for initializing and seeding the database
3. It implements CRUD operations for Products
4. It implements CRUD operations for Orders
5. It implements CRUD operations for Metals
6. It implements CRUD operations for Gemstones
7. It implements CRUD operations for Styles

This approach has several drawbacks:

- **Maintainability**: The file becomes difficult to navigate and understand
- **Testability**: Testing individual components becomes challenging
- **Collaboration**: Multiple developers working on different features may encounter merge conflicts
- **Single Responsibility Principle**: The class is handling too many responsibilities
- **Code Organization**: Related functionality is scattered throughout a large file

## The Solution: Inheritance and Specialization

A more maintainable approach is to use inheritance to create specialized database service classes for each resource type. This allows us to:

1. Keep common database functionality in a base class
2. Create specialized classes for each resource type
3. Organize code by resource rather than having everything in one file
4. Make the codebase more maintainable and testable

## Creating Specialized Database Services

You existing **DatabaseService** module will retain any methods that aren't resource-specific:

- Execution methods
- Database creation methods
- Database seeding methods

All other methods will be moved to specialized database service classes for each resource type:

#### Services/ProductService.cs

```csharp
public class ProductService : DatabaseService
{
    // Initialize the service by invoking the base class constructor
    public ProductService(IConfiguration config) : base(config) { }

    // Move all of the related Product database tasks here
}
```

#### Services/OrderService.cs

```cs
public class OrderService : DatabaseService
{
    public OrderService(IConfiguration config) : base(config) { }

    // Move all of the related Order database tasks here
}

```

Then make similar classes for `MetalService`, `GemstoneService`, and `StyleService`

## Updating Program.cs for Dependency Injection

To use our new specialized services, we need to update the dependency injection configuration in `Program.cs`:

```csharp
// Register the base DatabaseService for initialization
builder.Services.AddSingleton<DatabaseService>();

// Register specialized services
builder.Services.AddSingleton<ProductService>();
builder.Services.AddSingleton<OrderService>();
builder.Services.AddSingleton<MetalService>();
builder.Services.AddSingleton<GemstoneService>();
builder.Services.AddSingleton<StyleService>();

// Initialize the database
var dbService = app.Services.GetRequiredService<DatabaseService>();
await dbService.InitializeDatabaseAsync();
await dbService.SeedDatabaseAsync();
```

## Updating Endpoint Modules

Finally, we need to update our endpoint modules to use the specialized services:

```csharp
// ProductEndpoints.cs
public static class ProductEndpoints
{
    public static void MapProductEndpoints(this WebApplication app)
    {
        // Change DatabaseService to ProductService as the first parameter
        app.MapGet("/products", async (ProductService productService, string sortBy = null, string sortDirection = null) =>
        {
            var products = await productService.GetAllProductsAsync(sortBy, sortDirection);
            return Results.Ok(products);
        });
    }

    // Update all remaining endpoints
}
```

Now go to all other Endpoints modules and change the **DatabaseService** type to the corresponding new type you've created.

## Benefits of This Approach

This refactoring provides several benefits:

1. **Improved Organization**: Code is organized by resource type, making it easier to find and maintain
2. **Reduced File Size**: Each file is smaller and focused on a specific resource
3. **Better Testability**: Specialized services are easier to test in isolation
4. **Enhanced Collaboration**: Different team members can work on different services without conflicts
5. **Adherence to SOLID Principles**: Each class has a single responsibility
6. **Easier to Extend**: Adding new resource types is as simple as creating a new service class

## Considerations for Implementation

When implementing this pattern, consider the following:

1. **Base Class Design**: Carefully design the base class to include only truly common functionality
2. **Protected vs. Public**: Use protected access modifiers for methods that should only be used by derived classes
3. **Dependency Injection**: Ensure proper registration of all services in the dependency injection container
4. **Transaction Management**: Consider how transactions that span multiple services will be handled
5. **Testing Strategy**: Update your testing strategy to test both base and derived classes

## Conclusion

By refactoring our monolithic `DatabaseService` into a base class with specialized derived classes, we've improved the maintainability, testability, and organization of our codebase. This pattern allows us to adhere to the Single Responsibility Principle while still sharing common database functionality across all services.

In a real-world application, this approach scales much better as the number of resources and operations grows. Each resource type gets its own dedicated service class, keeping the codebase organized and manageable.

## Optional Bonus Exercise

Refactor your Harbor Master project to use the inheritance pattern described in this chapter:

1. Implement specialized service classes for Haulers, Ships, and Docks
2. Update your dependency injection configuration in `Program.cs`
3. Modify your endpoint modules to use the specialized services
4. Test your application to ensure everything still works as expected

This exercise will give you hands-on experience with inheritance and help you understand how it can be used to improve code organization and maintainability.

## Next Steps

The next chapter is an optional one that will challenge your prompt design skills with your favorite LLM to enhance your API with new functionality.

[Generate new features with an LLM >](./llm-guided-tasks.md)