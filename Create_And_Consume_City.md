# Lab Solution: City Table - Create and Consume API in Web API and MVC

## Overview
This lab solution demonstrates how to create a Web API for managing city data and consume the API in an ASP.NET Core MVC application. The solution includes:

- **Web API Project:** For managing CRUD operations for city data.
- **MVC Project:** For consuming the API and displaying city data in a user-friendly interface.

---

## Project Structure

### Web API Project
- **Controllers:**
  - `CityController.cs`: Handles API endpoints for city operations.
- **Models:**
  - `CityModel.cs`: Defines the structure of the city entity.
- **Data:**
  - `CityRepository`: Repository for handling city data operations.
- **Database:**
  - SQL Server database with a `City` table.
- **Program.cs:**
  - Configures services and middleware for the Web API project.
   ```csharp
   using DDL_API.Data;

   var builder = WebApplication.CreateBuilder(args);

   // Add services to the container.

   builder.Services.AddControllers();
   // Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
   builder.Services.AddEndpointsApiExplorer();
   builder.Services.AddSwaggerGen();
   builder.Services.AddScoped<CityRepository>();
   builder.Services.AddControllers();

   var app = builder.Build();

   // Configure the HTTP request pipeline.
   if (app.Environment.IsDevelopment())
   {
       app.UseSwagger();
       app.UseSwaggerUI();
   }

   app.UseHttpsRedirection();

   app.UseAuthorization();

   app.MapControllers();

   app.Run();
   ```
---

### **Dependency Injection Lifetime Methods**

1. **AddScoped**
   - **Purpose:** Registers a service with a scoped lifetime. A new instance of the service is created **once per HTTP request** and shared within that request.
   - **Use Case:** When you need the same instance of a service throughout a single request but don't want it to persist across requests.
   - **Example in Context:**
     ```csharp
     builder.Services.AddScoped<CityRepository>();
     ```
     Here, `CityRepository` is registered as a scoped service. During each HTTP request, a new instance of `CityRepository` will be created and reused wherever it's injected within that request.

2. **AddTransient**
   - **Purpose:** Registers a service with a transient lifetime. A new instance of the service is created **each time** it's requested.
   - **Use Case:** For lightweight, stateless services that do not hold any request-specific data.
   - **Example:**
     ```csharp
     builder.Services.AddTransient<IMyService, MyService>();
     ```

3. **AddSingleton**
   - **Purpose:** Registers a service with a singleton lifetime. A single instance of the service is created when the application starts and is shared across all requests.
   - **Use Case:** For services that need to maintain state globally or are expensive to initialize.
   - **Example:**
     ```csharp
     builder.Services.AddSingleton<ILogger, ConsoleLogger>();
     ```

---

### **Comparison of Lifetime Methods**
| **Method**       | **Lifetime**                  | **When to Use**                                                     |
|-------------------|-------------------------------|----------------------------------------------------------------------|
| **AddScoped**     | Per HTTP request              | For services tied to a single request (e.g., DB context).            |
| **AddTransient**  | Per service request           | For lightweight, stateless services.                                |
| **AddSingleton**  | Application-wide (single instance) | For stateful or expensive-to-create services.                       |

---

### **How to Use in Your Project**
In your `Program.cs`, you can register these services depending on their intended purpose. For example:
- **Repositories:** Typically scoped (`AddScoped`) as they often work with a single HTTP request (e.g., database operations).
- **Helper Classes:** Often transient (`AddTransient`) as they perform stateless operations.
- **Configuration/Settings:** Usually singleton (`AddSingleton`) as they remain constant throughout the application's lifecycle.

### MVC Project
- **Controllers:**
  - `CityController.cs`: Consumes the Web API for CRUD operations.
- **Views:**
  - `Index.cshtml`: Displays a list of cities.
  - `AddEdit.cshtml`: Form for adding and editing city data.
- **Models:**
  - `CityModel.cs`: View model for the city data.
---

## Setup Instructions

### Web API Project
1. Create a new ASP.NET Core Web API project.
2. Add a `CityModel` class for defining the city entity.
   
```csharp
namespace DDL_API.Models
{
    public class CityModel
    {
        public int? CityID { get; set; }
        public int StateID { get; set; }
        public int CountryID { get; set; }
        public string? StateName { get; set; }
        public string? CountryName { get; set; }
        public string CityName { get; set; }
        public string CityCode { get; set; }
    }

    public class CountryDropDownModel
    {
        public int CountryID { get; set; }
        public string CountryName { get; set; }
    }

    public class StateDropDownModel
    {
        public int StateID { get; set; }
        public string StateName { get; set; }
    }
}
 ```

3. Create a `CityController` with endpoints for CRUD operations.
 ```csharp
using DDL_API.Data;
using DDL_API.Models;
using Microsoft.AspNetCore.Mvc;

namespace DDL_API.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class CityController : Controller
    {
        private readonly CityRepository _cityRepository;

        public CityController(CityRepository cityRepository)
        {
            _cityRepository = cityRepository;
        }

        [HttpGet]
        public IActionResult GetAllCities()
        {
            var cities = _cityRepository.SelectAll();
            return Ok(cities);
        }

        [HttpGet("{id}")]
        public IActionResult GetCityById(int id)
        {
            var city = _cityRepository.SelectByPK(id);
            if (city == null)
            {
                return NotFound();
            }
            return Ok(city);
        }

        [HttpDelete("{id}")]
        public IActionResult DeleteCity(int id)
        {
            var isDeleted = _cityRepository.Delete(id);
            if (!isDeleted)
            {
                return NotFound();
            }
            return NoContent();
        }

        [HttpPost]
        public IActionResult InsertCity([FromBody] CityModel city)
        {
            if (city == null)
                return BadRequest();

            bool isInserted = _cityRepository.Insert(city);

            if (isInserted)
                return Ok(new { Message = "City inserted successfully!" });

            return StatusCode(500, "An error occurred while inserting the city.");
        }

        [HttpPut("{id}")]
        public IActionResult UpdateCity(int id, [FromBody] CityModel city)
        {
            if (city == null || id != city.CityID)
                return BadRequest();

            var isUpdated = _cityRepository.Update(city);
            if (!isUpdated)
                return NotFound();

            return NoContent();
        }

        [HttpGet("countries")]
        public IActionResult GetCountries()
        {
            var countries = _cityRepository.GetCountries();
            if (!countries.Any())
                return NotFound("No countries found.");

            return Ok(countries);
        }

        [HttpGet("states/{countryID}")]
        public IActionResult GetStatesByCountryID(int countryID)
        {
            if (countryID <= 0)
                return BadRequest("Invalid CountryID.");

            var states = _cityRepository.GetStatesByCountryID(countryID);
            if (!states.Any())
                return NotFound("No states found for the given CountryID.");

            return Ok(states);
        }
    }
}
   ```

4. Add a repository `CityRepository` for city operations.
```csharp
using DDL_API.Models;
using Microsoft.Data.SqlClient;
using System.Data;

namespace DDL_API.Data
{
    public class CityRepository
    {
        private readonly string _connectionString;

        public CityRepository(IConfiguration configuration)
        {
            _connectionString = configuration.GetConnectionString("DefaultConnection");
        }

        public IEnumerable<CityModel> SelectAll()
        {
            var cities = new List<CityModel>();
            using (SqlConnection conn = new SqlConnection(_connectionString))
            {
                SqlCommand cmd = new SqlCommand("PR_LOC_City_SelectAll", conn)
                {
                    CommandType = CommandType.StoredProcedure
                };
                conn.Open();
                SqlDataReader reader = cmd.ExecuteReader();
                while (reader.Read())
                {
                    cities.Add(new CityModel
                    {
                        CityID = Convert.ToInt32(reader["CityID"]),
                        StateID = Convert.ToInt32(reader["StateID"]),
                        CountryID = Convert.ToInt32(reader["CountryID"]),
                        CityName = reader["CityName"].ToString(),
                        StateName = reader["StateName"].ToString(),
                        CountryName = reader["CountryName"].ToString(),
                        CityCode = reader["CityCode"].ToString()
                    });
                }
            }
            return cities;
        }

        public CityModel SelectByPK(int cityID)
        {
            CityModel city = null;
            using (SqlConnection conn = new SqlConnection(_connectionString))
            {
                SqlCommand cmd = new SqlCommand("PR_LOC_City_SelectByPK", conn)
                {
                    CommandType = CommandType.StoredProcedure
                };
                cmd.Parameters.AddWithValue("@CityID", cityID);
                conn.Open();
                SqlDataReader reader = cmd.ExecuteReader();
                if (reader.Read())
                {
                    city = new CityModel
                    {
                        CityID = Convert.ToInt32(reader["CityID"]),
                        StateID = Convert.ToInt32(reader["StateID"]),
                        CountryID = Convert.ToInt32(reader["CountryID"]),
                        CityName = reader["CityName"].ToString(),
                        CityCode = reader["CityCode"].ToString()
                    };
                }
            }
            return city;
        }

        public bool Delete(int cityID)
        {
            using (SqlConnection conn = new SqlConnection(_connectionString))
            {
                SqlCommand cmd = new SqlCommand("PR_LOC_City_Delete", conn)
                {
                    CommandType = CommandType.StoredProcedure
                };
                cmd.Parameters.AddWithValue("@CityID", cityID);
                conn.Open();
                int rowsAffected = cmd.ExecuteNonQuery();
                return rowsAffected > 0;
            }
        }

        public bool Insert(CityModel city)
        {
            using (SqlConnection conn = new SqlConnection(_connectionString))
            {
                SqlCommand cmd = new SqlCommand("PR_LOC_City_Insert", conn)
                {
                    CommandType = CommandType.StoredProcedure
                };

                cmd.Parameters.AddWithValue("@StateID", city.StateID);
                cmd.Parameters.AddWithValue("@CountryID", city.CountryID);
                cmd.Parameters.AddWithValue("@CityName", city.CityName);
                cmd.Parameters.AddWithValue("@CityCode", city.CityCode);
                cmd.Parameters.AddWithValue("@Created", DateTime.Now); // Ensure @Created is provided
                cmd.Parameters.AddWithValue("@Modified", DateTime.Now); // Ensure @Modified is provided

                conn.Open();
                int rowsAffected = cmd.ExecuteNonQuery(); // Execute the stored procedure

                // Return true if the insertion was successful
                return rowsAffected > 0;
            }
        }

        public bool Update(CityModel city)
        {
            using (SqlConnection conn = new SqlConnection(_connectionString))
            {
                SqlCommand cmd = new SqlCommand("PR_LOC_City_Update", conn)
                {
                    CommandType = CommandType.StoredProcedure
                };
                cmd.Parameters.AddWithValue("@CityID", city.CityID);
                cmd.Parameters.AddWithValue("@StateID", city.StateID);
                cmd.Parameters.AddWithValue("@CountryID", city.CountryID); // New parameter
                cmd.Parameters.AddWithValue("@CityName", city.CityName);
                cmd.Parameters.AddWithValue("@CityCode", city.CityCode);
                cmd.Parameters.Add("@Modified", SqlDbType.DateTime).Value = DBNull.Value;

                conn.Open();
                int rowsAffected = cmd.ExecuteNonQuery();
                return rowsAffected > 0;
            }
        }

        public IEnumerable<CountryDropDownModel> GetCountries()
        {
            var countries = new List<CountryDropDownModel>();

            using (SqlConnection conn = new SqlConnection(_connectionString))
            {
                SqlCommand cmd = new SqlCommand("PR_LOC_Country_SelectComboBox", conn)
                {
                    CommandType = CommandType.StoredProcedure
                };

                conn.Open();
                SqlDataReader reader = cmd.ExecuteReader();

                while (reader.Read())
                {
                    countries.Add(new CountryDropDownModel
                    {
                        CountryID = Convert.ToInt32(reader["CountryID"]),
                        CountryName = reader["CountryName"].ToString()
                    });
                }
            }

            return countries;
        }

        public IEnumerable<StateDropDownModel> GetStatesByCountryID(int countryID)
        {
            var states = new List<StateDropDownModel>();

            using (SqlConnection conn = new SqlConnection(_connectionString))
            {
                SqlCommand cmd = new SqlCommand("PR_LOC_State_SelectComboBoxByCountryID", conn)
                {
                    CommandType = CommandType.StoredProcedure
                };
                cmd.Parameters.AddWithValue("@CountryID", countryID);

                conn.Open();
                SqlDataReader reader = cmd.ExecuteReader();

                while (reader.Read())
                {
                    states.Add(new StateDropDownModel
                    {
                        StateID = Convert.ToInt32(reader["StateID"]),
                        StateName = reader["StateName"].ToString()
                    });
                }
            }

            return states;
        }
    }
}
   ```


5. Test the API endpoints using Postman or Swagger.

### MVC Project
1. Create a new ASP.NET Core MVC project.
2. Add a `CityController` for consuming the API.
```csharp
using DDL_MVC.Models;
using Microsoft.AspNetCore.Mvc;
using Newtonsoft.Json;
using System.Text;

namespace DDL_MVC.Controllers
{
    public class CityController : Controller
    {
        private readonly IConfiguration _configuration;
        private readonly HttpClient _httpClient;

        public CityController(IConfiguration configuration)
        {
            _configuration = configuration;
            _httpClient = new HttpClient
            {
                BaseAddress = new System.Uri(_configuration["WebApiBaseUrl"])
            };
        }

        public async Task<IActionResult> Index()
        {
            var response = await _httpClient.GetAsync("api/city");
            if (response.IsSuccessStatusCode)
            {
                var data = await response.Content.ReadAsStringAsync();
                var cities = JsonConvert.DeserializeObject<List<CityModel>>(data);
                return View(cities);
            }
            return View(new List<CityModel>());
        }

        public async Task<IActionResult> Add(int? CityID)
        {
            await LoadCountryList();
            if (CityID.HasValue)
            {
                var response = await _httpClient.GetAsync($"api/city/{CityID}");
                if (response.IsSuccessStatusCode)
                {
                    var data = await response.Content.ReadAsStringAsync();
                    Console.Write(data);                    var city = JsonConvert.DeserializeObject<CityModel>(data);
                    ViewBag.StateList = await GetStatesByCountryID(city.CountryID);
                    return View(city);
                }
            }
            return View(new CityModel());
        }

        [HttpPost]
        public async Task<IActionResult> Save(CityModel city)
        {
            if (ModelState.IsValid)
            {
                var json = JsonConvert.SerializeObject(city);
                var content = new StringContent(json, Encoding.UTF8, "application/json");
                HttpResponseMessage response;

                if (city.CityID == null)
                    response = await _httpClient.PostAsync("api/city", content);
                else
                    response = await _httpClient.PutAsync($"api/city/{city.CityID}", content);

                if (response.IsSuccessStatusCode)
                    return RedirectToAction("Index");
                var errorContent = await response.Content.ReadAsStringAsync();
                Console.WriteLine($"Error Response: {errorContent}");
                TempData["Error"] = errorContent;
            }
            await LoadCountryList();
            return View("Add", city);
        }

        public async Task<IActionResult> Delete(int CityID)
        {
            var response = await _httpClient.DeleteAsync($"api/city/{CityID}");
            return RedirectToAction("Index");
        }

        private async Task LoadCountryList()
        {
            var response = await _httpClient.GetAsync("api/city/countries");
            if (response.IsSuccessStatusCode)
            {
                var data = await response.Content.ReadAsStringAsync();
                var countries = JsonConvert.DeserializeObject<List<CountryDropDownModel>>(data);
                ViewBag.CountryList = countries;
            }
        }

        [HttpPost]
        public async Task<JsonResult> GetStatesByCountry(int CountryID)
        {
            var states = await GetStatesByCountryID(CountryID);
            return Json(states);
        }

        private async Task<List<StateDropDownModel>> GetStatesByCountryID(int CountryID)
        {
            var response = await _httpClient.GetAsync($"api/city/states/{CountryID}");
            if (response.IsSuccessStatusCode)
            {
                var data = await response.Content.ReadAsStringAsync();
                return JsonConvert.DeserializeObject<List<StateDropDownModel>>(data);
            }
            return new List<StateDropDownModel>();
        }
    }
}

   ```

3. Create views for listing, adding, and editing cities.
```html
   <!-- Code for Index.cshtml -->
  @model IEnumerable<DDL_MVC.Models.CityModel>
<h2>City List</h2>
<table>
    <thead>
        <tr>
            <th>City Name</th>
            <th>State Name</th>
            <th>Country Name</th>
            <th>Actions</th>
        </tr>
    </thead>
    <tbody>
        @foreach (var city in Model)
        {
            <tr>
                <td>@city.CityName</td>
                <td>@city.StateName</td>
                <td>@city.CountryName</td>
                <td>
                    <a asp-action="Add" asp-route-CityID="@city.CityID">Edit</a>
                    <form asp-action="Delete"
                          asp-route-CityID="@city.CityID"
                          method="post">
                        <button type="submit">Delete</button>
                    </form>
                </td>
            </tr>
        }
    </tbody>
</table></Lab14.City.Models.CityModel>
```
```html
<!-- Code for AddEdit.cshtml -->
@model DDL_MVC.Models.CityModel

<h2>Add/Edit City</h2>
<form method="post" asp-controller="City" asp-action="Save">
    <input type="hidden" asp-for="CityID" />

    <label>Country</label>
    <select asp-for="CountryID" asp-items="@(new SelectList(ViewBag.CountryList, "CountryID", "CountryName"))">
        <option value="">Select Country</option>
    </select>

    <label>State</label>
    <select asp-for="StateID" asp-items="@(new SelectList(ViewBag.StateList ?? new List<StateDropDownModel>(), "StateID", "StateName", Model.StateID))"></select>

    <label>City Name</label>
    <input asp-for="CityName" />

    <label>City Code</label>
    <input asp-for="CityCode" />

    <button type="submit">Save</button>
</form>

@section Scripts {
    <script>
        $('#CountryID').change(function () {
            $.post('@Url.Action("GetStatesByCountry")', { CountryID: $(this).val() }, function (data) {
                console.log("Data received from server:", data);

                // Clear the State dropdown and add the default option
                $('#StateID').empty().append('<option value="">Select State</option>');

                // Populate the dropdown with states from the response
                $.each(data, function (i, state) {
                    $('#StateID').append(`<option value="${state.stateID}">${state.stateName}</option>`);
                });
            }).fail(function (xhr, status, error) {
                console.error("Error fetching states:", error);
            });
        });

    </script>
}
   ```

---

## Features

### Web API
- Create, read, update, and delete city data.
- Use of repository pattern for data access.

### MVC Application
- Consume the Web API for managing city data.
- Cascading dropdowns for country and state selection.

---

## Usage

### Running the Solution
1. Run the Web API project and ensure the API is accessible.
2. Update the MVC project `appsettings.json` with the correct API base URL.
3. Run the MVC project to interact with city data.
