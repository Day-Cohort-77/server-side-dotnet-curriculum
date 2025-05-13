# Dependency Injection and Model Binding

In your .NET Web API endpoint, the parameters `Ship ship` and `DatabaseService db` are automatically populated through two different mechanisms in ASP.NET Core:

## For `DatabaseService db`: Dependency Injection

This is handled by ASP.NET Core's built-in dependency injection (DI) system:

```cs
// In Program.cs
builder.Services.AddSingleton<DatabaseService>();
```


The process works as follows:

**Service Registration**: In Program.cs (line 7), the DatabaseService is registered as a singleton in the DI container.

**Service Resolution**: When the endpoint is called, the DI container automatically:

- Identifies that the endpoint requires a DatabaseService parameter
- Resolves the registered singleton instance
- Provides that instance to the endpoint method

## For `Ship ship`: Model Binding

This is handled by ASP.NET Core's model binding system:

```cs
app.MapPost("/ships", async (Ship ship, DatabaseService db) => { ... })
```

The process works as follows:

**Request Parsing**: When a POST request is made to "/ships" with a JSON body, the ASP.NET Core pipeline:

- Reads the request body
- Identifies the content type (typically application/json)
- Deserializes the JSON payload into a Ship object

**Property Mapping**: The JSON properties are mapped to the corresponding properties in the Ship class:

```cs
public class Ship
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public string Type { get; set; } = string.Empty;
    public int? DockId { get; set; }
}
```


**Parameter Binding**: The fully populated Ship object is then provided as a parameter to the endpoint method.

## Summary

This is part of what makes Minimal APIs in .NET so powerful and concise:

**Dependency Injection**: Services like DatabaseService are automatically provided by the DI container

**Model Binding**: Request data is automatically deserialized into model objects like Ship

You don't need to manually extract data from the request or resolve services - the framework handles this automatically, allowing you to focus on your business logic.

