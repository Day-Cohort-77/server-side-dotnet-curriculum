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

First, implement the `UpdateProductAsync` method in your `DatabaseService` class

```csharp
// Update a product
public async Task<Product> UpdateProductAsync(Product product)
{

}
```

## Implementing the Basic Endpoint

Continue by implementing a simple endpoint that updates a product. You likely have a `Endpoints/ProductEndpoints.cs` module — or something similar. Add the following endpoint to that module.

```csharp
// PUT /products/{id} - Update a product
app.MapPut("/products/{id}", async (int id, Product updatedProduct, DatabaseService db) =>
{

});
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

1. Restart your debugger
2. Send a PUT request to the `/products/{id}` endpoint and click on it
    - Enter a product ID and a JSON request body with the updated product data:
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
3. You should see a 204 response with the updated product details

## Conclusion

In this chapter, you've learned how to implement a PUT endpoint to update a product in the Jewelry Junction API. You've created a route handler that validates the input data, updates the product in the database, and returns the updated product.

This is a fundamental operation in RESTful APIs, and you'll use similar patterns for updating other resources in your applications.
