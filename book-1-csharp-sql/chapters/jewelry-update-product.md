# Update a Product

In this chapter, we'll implement the endpoint for updating a product in our Jewelry Junction API. This will be our first experience with PUT operations, which allow us to update existing resources.

## Learning Objectives

By the end of this chapter, you should be able to:
- Implement a PUT endpoint to update a product
- Validate the product data before updating
- Update a product in the database
- Return appropriate responses

## Understanding PUT Requests

In REST APIs, PUT requests are used to update existing resources. When we make a PUT request, we're asking the server to replace the current version of a resource with the new version we're providing.

For our product update endpoint, we'll:
1. Receive a PUT request with the product ID in the URL and the updated product data in the request body
2. Check if the product exists
3. Update the product in the database
4. Return the updated product

## Implementing the Database Method

First, implement the `UpdateProductAsync` method in your `DatabaseService` class:

```csharp
// Update a product
public async Task<Product> UpdateProductAsync(Product product)
{
    using var connection = CreateConnection();
    await connection.OpenAsync();

    // Create the SQL command to update the product
    using var command = new NpgsqlCommand(
        @"UPDATE products
                   SET name = @name,
                       description = @description,
                       price = @price,
                       metal_id = @metalId,
                       gemstone_id = @gemstoneId,
                       style_id = @styleId
                   WHERE id = @id",
        connection);

    // Add parameters to the command
    command.Parameters.AddWithValue("@id", product.Id);
    command.Parameters.AddWithValue("@name", product.Name);
    command.Parameters.AddWithValue("@description", product.Description);
    command.Parameters.AddWithValue("@price", product.Price);
    command.Parameters.AddWithValue("@metalId", product.MetalId.Value);
    command.Parameters.AddWithValue("@gemstoneId", product.GemstoneId.Value);
    command.Parameters.AddWithValue("@styleId", product.StyleId.Value);

    // Execute the command
    await command.ExecuteNonQueryAsync();

    // Retrieve and return the updated product
    return await GetProductByIdAsync(product.Id);
}
```

## Implementing the Basic Endpoint

Continue by implementing a simple endpoint that updates a product. You likely have a `Endpoints/ProductEndpoints.cs` module — or something similar. Add the following endpoint to that module.

```csharp
// PUT /products/{id} - Update a product
app.MapPut("/products/{id}", async (int id, Product updatedProduct, DatabaseService db) =>
{
    try
    {
        // Check if the product exists
        var existingProduct = await db.GetProductByIdAsync(id);
        if (existingProduct == null)
        {
            return Results.NotFound($"Product with ID {id} not found");
        }

        // Set the ID from the route parameter
        updatedProduct.Id = id;

        // Update the product
        var result = await db.UpdateProductAsync(updatedProduct);

        // Signal success with a 204 status code on the response
        return Results.NoContent();
    }
    catch (Exception ex)
    {
        return Results.Problem($"An error occurred while updating the product: {ex.Message}");
    }
})
.WithName("UpdateProduct")
.WithOpenApi();
```

This endpoint:
- Maps to PUT requests at the `/products/{id}` URL
- Takes the product ID from the URL and the updated product data from the request body
- Checks if the product exists
- Updates the product using our `DatabaseService`

## Adding Basic Validation

Before updating a product, we should validate the data to ensure it's complete and valid. Let's add some basic validation to our endpoint. Add the following code after you check if the product exists, and before you set the `Id` property.

```csharp
// Validate required fields
if (string.IsNullOrWhiteSpace(updatedProduct.Name))
{
   return Results.BadRequest("Name is required");
}

if (string.IsNullOrWhiteSpace(updatedProduct.Description))
{
   return Results.BadRequest("Description is required");
}

// Validate numeric values
if (updatedProduct.Price <= 0)
{
   return Results.BadRequest("Price must be greater than zero");
}
```

This validation:
1. Checks that required fields (name, description) are provided
2. Validates numeric values (price > 0, stock quantity >= 0)

## Testing the Endpoint

Now that we've implemented our PUT endpoint, let's test it:

1. Open Swagger at `https://localhost:7042/swagger` (or the URL shown in your terminal)
2. Find the PUT `/products/{id}` endpoint and click on it
3. Click the "Try it out" button
4. Enter a product ID and a JSON request body with the updated product data:
   ```json
   {
     "name": "Updated Diamond Ring",
     "description": "A beautiful diamond ring with an updated design",
     "price": 1299.99,
     "metalId": 1,
     "gemstoneId": 1,
     "styleId": 2
   }
   ```
5. Click the "Execute" button
6. You should see a 204 response with the updated product details

## Conclusion

In this chapter, you've learned how to implement a PUT endpoint to update a product in the Jewelry Junction API. You've created a route handler that validates the input data, updates the product in the database, and returns the updated product.

This is a fundamental operation in RESTful APIs, and you'll use similar patterns for updating other resources in your applications.

## Optional Practice Exercise

Enhance your product update functionality by:

1. Adding validation for foreign keys (check if metal, gemstone, and style exist)
2. Adding the ability to update multiple products at once
3. Adding support for partial updates (only updating specific fields)