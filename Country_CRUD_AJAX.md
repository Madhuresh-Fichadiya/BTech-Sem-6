# Country Management MVC Application

This project is a simple ASP.NET Core MVC application to manage countries. It includes functionalities to view, add, edit, and delete countries through a dynamic user interface.

## Features

1. **Country Listing**: Displays a table of all countries.
2. **Add/Edit Modal**: A modal to add or update country details dynamically.
3. **AJAX-Based Operations**: All CRUD operations (Create, Read, Update, Delete) are performed asynchronously using jQuery and AJAX.
4. **Web API Integration**: The application consumes a RESTful API for country management.

## Project Code

### Model:
```csharp
public class CountryModel
{
    public int CountryID { get; set; }
    public string CountryName { get; set; }
    public string CountryCode { get; set; }
}
```

---
### Countroller Code:
```csharp
using DDL_MVC.Models;
using Microsoft.AspNetCore.Mvc;
using Newtonsoft.Json;
using System.Text;

namespace DDL_MVC.Controllers
{
    public class CountryController : Controller
    {
        private readonly IConfiguration _configuration;
        private readonly HttpClient _httpClient;

        public CountryController(IConfiguration configuration)
        {
            _configuration = configuration;
            _httpClient = new HttpClient
            {
                BaseAddress = new System.Uri(_configuration["WebApiBaseUrl"])
            };
            _httpClient.DefaultRequestHeaders.Add("Accept", "application/json");
        }


        public IActionResult Index()
        {
            return View();
        }

        [HttpGet]
        public async Task<JsonResult> GetCountries()
        {
            var response = await _httpClient.GetAsync("api/country");
            if (response.IsSuccessStatusCode)
            {
                var data = await response.Content.ReadAsStringAsync();
                var countries = JsonConvert.DeserializeObject<List<CountryModel>>(data);
                return Json(countries);
            }
            return Json(new List<CountryModel>());
        }

        [HttpPost]
        public async Task<JsonResult> SaveCountry([FromBody] CountryModel country)
        {
            HttpResponseMessage response;

            if (country.CountryID > 0) 
            {
                var jsonContent = JsonConvert.SerializeObject(country);
                var content = new StringContent(jsonContent, Encoding.UTF8, "application/json");
                response = await _httpClient.PutAsync($"api/country/{country.CountryID}", content);
            }
            else 
            {
                var jsonContent = JsonConvert.SerializeObject(country);
                var content = new StringContent(jsonContent, Encoding.UTF8, "application/json");
                response = await _httpClient.PostAsync($"api/country", content);
            }

            return Json(response.IsSuccessStatusCode);
        }


        [HttpDelete]
        public async Task<JsonResult> DeleteCountry(int id)
        {
            var response = await _httpClient.DeleteAsync($"api/country/{id}");
            return Json(response.IsSuccessStatusCode);
        }

    }
}
```

### Index Page
```html
@model IEnumerable<CountryModel>
@{
    ViewData["Title"] = "Country List";
}

<h2>Country List</h2>
<button class="btn btn-primary mb-2" onclick="showModal()">Add Country</button>
<table id="countriesTable" class="table table-bordered">
    <thead>
        <tr>
            <th>Country ID</th>
            <th>Country Name</th>
            <th>Country Code</th>
            <th>Actions</th>
        </tr>
    </thead>
    <tbody>
        <!-- Rows will be populated here dynamically -->
    </tbody>
</table>

<!-- Add/Edit Modal -->
<div id="countryModal" class="modal fade" tabindex="-1" role="dialog">
    <div class="modal-dialog" role="document">
        <div class="modal-content">
            <div class="modal-header">
                <h5 class="modal-title">Add/Edit Country</h5>
                <button type="button" class="close" data-bs-dismiss="modal" aria-label="Close">
                    <span aria-hidden="true">&times;</span>
                </button>
            </div>
            <div class="modal-body">
                <form id="countryForm">
                    <input type="hidden" id="countryID" />
                    <div class="form-group">
                        <label for="countryName">Country Name</label>
                        <input type="text" id="countryName" class="form-control" required />
                    </div>
                    <div class="form-group">
                        <label for="countryCode">Country Code</label>
                        <input type="text" id="countryCode" class="form-control" required />
                    </div>
                </form>
            </div>
            <div class="modal-footer">
                <button type="button" class="btn btn-primary" id="saveCountryButton">Save</button>
                <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Close</button>
            </div>
        </div>
    </div>
</div>

@section Scripts {
    <script src="https://code.jquery.com/jquery-3.7.1.js"></script>
    <script>
        $(document).ready(function () {
            loadCountries();
        });

        function showModal(id = '', name = '', code = '') {
            $("#countryID").val(id);
            $("#countryName").val(name); 
            $("#countryCode").val(code); 
            $("#countryModal").modal('show'); 
        }

        function loadCountries() {
            $.ajax({
                url: '/Country/GetCountries',
                type: 'GET',
                success: function (data) {
                    const tableBody = $("#countriesTable tbody");
                    tableBody.empty();
                    data.forEach(country => {
                        console.log(country);
                        tableBody.append(`
                            <tr>
                                <td>${country.countryID}</td>
                                <td>${country.countryName}</td>
                                <td>${country.countryCode}</td>
                                <td>
                                    <button class="btn btn-warning btn-sm" onclick="showModal(${country.countryID}, '${country.countryName}', '${country.countryCode}')">Edit</button>
                                    <button class="btn btn-danger btn-sm" onclick="deleteCountry(${country.countryID})">Delete</button>
                                </td>
                            </tr>
                        `);
                    });
                }
            });
        }

        $("#saveCountryButton").click(function () {
            const country = {
                countryID: $("#countryID").val() || 0,
                countryName: $("#countryName").val(),
                countryCode: $("#countryCode").val()
            };

            if (!country.countryName || !country.countryCode) {
                alert("Please fill all fields.");
                return;
            }

            $.ajax({
                url: '/Country/SaveCountry',
                type: 'POST',
                data: JSON.stringify(country),
                contentType: 'application/json',
                success: function () {
                    $("#countryModal").modal('hide');
                    loadCountries();
                },
                error: function () {
                    alert("Failed to save the country. Please try again.");
                }
            });
        });

        function deleteCountry(id) {
            if (confirm("Are you sure you want to delete this country?")) {
                $.ajax({
                    url: '/Country/DeleteCountry',
                    type: 'DELETE',
                    data: { id: id },
                    success: function () {
                        loadCountries();
                    }
                });
            }
        }
    </script>
}
```

---
