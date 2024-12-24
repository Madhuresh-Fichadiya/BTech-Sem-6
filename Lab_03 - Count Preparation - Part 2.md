# Count Preparation - Part 2

## Overview
This guide provides step-by-step instructions to implement functionality for retrieving cities based on a specific state.

---

## Step 1: Add StateID Parameter in CityController Index Method
Update the Index method in the CityController to accept a StateID parameter and filter the city list accordingly.

### Code Changes:
*CityController.cs*
```csharp
public async Task<IActionResult> Index(int? StateID)
{
    List<CityModel> cities = new List<CityModel>();
    List<CityModel> newCities = new List<CityModel>();

    try
    {
        // Make the HTTP GET request
        HttpResponseMessage response = await _client.GetAsync("City/");

        if (response.IsSuccessStatusCode)
        {
            // Read the response content
            string data = await response.Content.ReadAsStringAsync();

            // Directly deserialize the JSON array into a List<CityModel>
            newCities = JsonConvert.DeserializeObject<List<CityModel>>(data);
            if (StateID.HasValue)
            {
                foreach (var item in newCities)
                {
                    if (item.StateID == StateID)
                    {
                        cities.Add(item);
                    }
                }
            }
            else
            {
                cities = newCities;
            }
        }
        else
        {
            // Handle unsuccessful responses
            Console.WriteLine($"API Error: {response.StatusCode}");
        }
    }
    catch (Exception ex)
    {
        // Log or handle exceptions
        Console.WriteLine($"Exception occurred: {ex.Message}");
    }
    Console.WriteLine(cities);
    return View(cities);
}
```

---

## Step 2: Add Link to State List Page
Add a link in the state list page to navigate to the city list page filtered by StateID.

### Code Changes:
*StateList.cshtml*
html
```<table class="table table-striped">
    <thead>
        <tr>
            <th>State ID</th>
            <th>State Name</th>
            <th>State Code</th>
            <th>Country ID</th>
            <th>Country Name</th>
            <th>City Count</th>
        </tr>
    </thead>
    <tbody>
        @foreach (var city in Model)
        {
            <tr>
                <td>@city.StateID</td>
                <td>@city.StateName</td>
                <td>@city.StateCode</td>
                <td>@city.CountryID</td>
                <td>@city.CountryName</td>
                <td><a asp-action="Index" asp-controller="City" asp-route-StateID=@city.StateID>@city.CityCount</a></td>
            </tr>
        }
    </tbody>
</table>
```
