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

## Implementing the Basic Endpoint

Let's start by implementing a simple endpoint that updates a product. We'll add this to our `Program.cs` file:

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

        // Return the updated product
        return Results.Ok(result);
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
- Returns the updated product

## Implementing the Database Method

Now, let's implement the `UpdateProductAsync` method in our `DatabaseService` class:

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
              stock_quantity = @stockQuantity,
              metal_id = @metalId,
              category_id = @categoryId,
              discount_id = @discountId
          WHERE id = @id",
        connection);

    // Add parameters to the command
    command.Parameters.AddWithValue("@id", product.Id);
    command.Parameters.AddWithValue("@name", product.Name);
    command.Parameters.AddWithValue("@description", product.Description);
    command.Parameters.AddWithValue("@price", product.Price);
    command.Parameters.AddWithValue("@stockQuantity", product.StockQuantity);

    // Handle nullable foreign keys
    if (product.MetalId.HasValue)
    {
        command.Parameters.AddWithValue("@metalId", product.MetalId.Value);
    }
    else
    {
        command.Parameters.AddWithValue("@metalId", DBNull.Value);
    }

    if (product.CategoryId.HasValue)
    {
        command.Parameters.AddWithValue("@categoryId", product.CategoryId.Value);
    }
    else
    {
        command.Parameters.AddWithValue("@categoryId", DBNull.Value);
    }

    if (product.DiscountId.HasValue)
    {
        command.Parameters.AddWithValue("@discountId", product.DiscountId.Value);
    }
    else
    {
        command.Parameters.AddWithValue("@discountId", DBNull.Value);
    }

    // Execute the command
    await command.ExecuteNonQueryAsync();

    // Retrieve and return the updated product
    return await GetProductByIdAsync(product.Id);
}
```

This method:
1. Creates a database connection
2. Prepares an UPDATE SQL statement with parameters
3. Sets parameter values from the product object
4. Handles nullable foreign keys by using DBNull.Value when appropriate
5. Executes the update command
6. Retrieves and returns the updated product

## Adding Basic Validation

Before updating a product, we should validate the data to ensure it's complete and valid. Let's add some basic validation to our endpoint:

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

        if (updatedProduct.StockQuantity < 0)
        {
            return Results.BadRequest("Stock quantity cannot be negative");
        }

        // Set the ID from the route parameter
        updatedProduct.Id = id;

        // Update the product
        var result = await db.UpdateProductAsync(updatedProduct);

        // Return the updated product
        return Results.Ok(result);
    }
    catch (Exception ex)
    {
        return Results.Problem($"An error occurred while updating the product: {ex.Message}");
    }
})
.WithName("UpdateProduct")
.WithOpenApi();
```

This validation:
1. Checks that required fields (name, description) are provided
2. Validates numeric values (price > 0, stock quantity >= 0)

## Testing the Endpoint

Now that we've implemented our PUT endpoint, let's test it:

1. Start the API:
   ```bash
   dotnet run
   ```

2. Open Swagger at `https://localhost:7042/swagger` (or the URL shown in your terminal)

3. Find the PUT `/products/{id}` endpoint and click on it

4. Click the "Try it out" button

5. Enter a product ID and a JSON request body with the updated product data:
   ```json
   {
     "name": "Updated Diamond Ring",
     "description": "A beautiful diamond ring with an updated design",
     "price": 1299.99,
     "stockQuantity": 5,
     "metalId": 1,
     "categoryId": 1,
     "discountId": null
   }
   ```

6. Click the "Execute" button

7. You should see a 200 OK response with the updated product details

## Understanding HTTP Status Codes

Our endpoint returns different HTTP status codes depending on the result:

- 200 OK: The product was updated successfully
- 400 Bad Request: The request data is invalid (e.g., missing required fields)
- 404 Not Found: The product with the specified ID doesn't exist
- 500 Internal Server Error: An unexpected error occurred

These status codes help the client understand what happened with their request.

## Conclusion

In this chapter, you've learned how to implement a PUT endpoint to update a product in the Jewelry Junction API. You've created a route handler that validates the input data, updates the product in the database, and returns the updated product.

This is a fundamental operation in RESTful APIs, and you'll use similar patterns for updating other resources in your applications.

## Practice Exercise

Enhance your product update functionality by:
1. Adding validation for foreign keys (check if metal, category, and discount exist)
2. Implementing a simple logging mechanism to track product updates
3. Adding the ability to update a product's gemstones
4. Creating a simple client interface to test your PUT endpoint