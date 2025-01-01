# Consuming API in ASP.NET Core MVC

## AddCountry Action
The `AddCountry` action is responsible for fetching country details when a `CountryID` is provided. If no `CountryID` is specified, it returns an empty model for creating a new country.

### Key Steps:
1. **Check if `CountryID` is provided:**
   ```csharp
   if (CountryID.HasValue)
   ```
   
2. **Make an HTTP GET request to fetch the country details:**
   ```csharp
   var response = await _httpClient.GetAsync($"api/Country/{CountryID}");
   ```

3. **Deserialize the JSON response:**
   ```csharp
   var country = JsonConvert.DeserializeObject<CountryModel>(data);
   ```

4. **Return the view with the fetched data:**
   ```csharp
   return View(country);
   ```

5. **Handle errors and exceptions:**
   ```csharp
   Console.WriteLine($"Error Response: {errorContent}");
   Console.WriteLine($"Exception: {ex.Message}");
   ```

If no `CountryID` is provided, the method returns:
```csharp
return View(new CountryModel());
```

### AddCountry Action Code:
```csharp
public async Task<IActionResult> AddCountry(int? CountryID)
{
    if (CountryID.HasValue)
    {
        try
        {
            var response = await _httpClient.GetAsync($"api/Country/{CountryID}");

            if (response.IsSuccessStatusCode)
            {
                var data = await response.Content.ReadAsStringAsync();
                var country = JsonConvert.DeserializeObject<CountryModel>(data);
                return View(country);
            }

            var errorContent = await response.Content.ReadAsStringAsync();
            Console.WriteLine($"Error Response: {errorContent}");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Exception: {ex.Message}");
        }
    }

    return View(new CountryModel());
}
```

## CountrySave Action
The `CountrySave` action handles both creating and updating country data using the HTTP methods POST and PUT.

### Key Steps:
1. **Check if the model state is valid:**
   ```csharp
   if (ModelState.IsValid)
   ```

2. **Serialize the `CountryModel` object to JSON:**
   ```csharp
   var json = JsonConvert.SerializeObject(country);
   ```

3. **Create the HTTP content:**
   ```csharp
   var content = new StringContent(json, Encoding.UTF8, "application/json");
   ```

4. **Determine POST or PUT based on `CountryID`:**
   - **POST (Create):**
     ```csharp
     response = await _httpClient.PostAsync("api/Country", content);
     ```
   - **PUT (Update):**
     ```csharp
     response = await _httpClient.PutAsync($"api/Country/{country.CountryID}", content);
     ```

5. **Redirect to the country list on success:**
   ```csharp
   if (response.IsSuccessStatusCode)
       return RedirectToAction("CountryList");
   ```

6. **Handle errors and log exceptions:**
   ```csharp
   Console.WriteLine($"Error Response: {errorContent}");
   TempData["Error"] = errorContent;
   Console.WriteLine($"Exception: {ex.Message}");
   ```

7. **Return the AddCountry view with the current model on failure:**
   ```csharp
   return View("AddCountry", country);
   ```

### CountrySave Action Code:
```csharp
[HttpPost]
public async Task<IActionResult> CountrySave(CountryModel country)
{
    if (ModelState.IsValid)
    {
        try
        {
            var json = JsonConvert.SerializeObject(country);
            var content = new StringContent(json, Encoding.UTF8, "application/json");
            HttpResponseMessage response;

            if (country.CountryID == null)
            {
                response = await _httpClient.PostAsync("api/Country", content);
            }
            else
            {
                response = await _httpClient.PutAsync($"api/Country/{country.CountryID}", content);
            }

            if (response.IsSuccessStatusCode)
                return RedirectToAction("CountryList");

            var errorContent = await response.Content.ReadAsStringAsync();
            Console.WriteLine($"Error Response: {errorContent}");
            TempData["Error"] = errorContent;
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Exception: {ex.Message}");
        }
    }

    return View("AddCountry", country);
}
```

## Debugging in Consuming API
To ensure smooth consumption of APIs, debug the following steps:

1. **Check API Endpoint:**
   Verify that the URL provided in `GetAsync`, `PostAsync`, and `PutAsync` calls is correct.
   ```csharp
   Console.WriteLine($"Request URL: api/Country/{CountryID}");
   ```

2. **Inspect HTTP Response:**
   Log the response status code and content for analysis.
   ```csharp
   Console.WriteLine($"Response Status Code: {response.StatusCode}");
   Console.WriteLine($"Response Content: {errorContent}");
   ```

3. **Validate Model Data:**
   Ensure the `CountryModel` passed to the API has all required fields filled.
   ```csharp
   Console.WriteLine(JsonConvert.SerializeObject(country));
   ```

4. **Handle Exceptions Gracefully:**
   Catch exceptions and log detailed error messages for debugging.
   ```csharp
   catch (Exception ex)
   {
       Console.WriteLine($"Exception: {ex.Message}");
   }
   ```

5. **Use Developer Tools:**
   - Inspect network requests in browser developer tools.
   - Use tools like Postman to manually test API endpoints.
