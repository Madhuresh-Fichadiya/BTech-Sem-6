**CRUD operations for the City table by consuming a Web API** in ASP.NET Core MVC:

---

## Steps to Implement

### 1. **Setup Project and Install Required Packages**

- **Create a new ASP.NET Core MVC project**.
- Install the following NuGet packages:
  ```bash
  Install-Package Newtonsoft.Json
  Install-Package Microsoft.Extensions.Configuration
  ```
- Ensure your project has the **HttpClientFactory** setup.

---

### 2. **Configure API Base URL**

- Open `appsettings.json` and add the API base URL:
  ```json
  {
    "WebApiBaseUrl": "https://localhost:7004/"
  }
  ```
- Ensure the API project is running on this URL or modify it accordingly.

---

### 3. **Create Models**

- Create a folder named `Models` and add the required model classes.

**CityModel.cs**

```csharp
namespace Lab14.City.Models
{
    public class CityModel
    {
        public int? CityID { get; set; }
        public string CityName { get; set; }
        public string CityCode { get; set; }
        public int CountryID { get; set; }
        public int StateID { get; set; }
        public string StateName { get; set; }
        public string CountryName { get; set; }
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

---

### 4. **Create CityController**

- Add a controller named `CityController` and implement CRUD operations as shown below:

**Controller Code**

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Configuration;
using Newtonsoft.Json;
using System.Collections.Generic;
using System.Net.Http;
using System.Text;
using System.Threading.Tasks;
using Lab14.City.Models;

namespace Lab14.Areas.City.Controllers
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
                    var city = JsonConvert.DeserializeObject<CityModel>(data);
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

---

### 5. **Create Views**

- Add `City` folder in the `Views` directory and create the following views.

**Index.cshtml**

```html
@model IEnumerable<Lab14.City.Models.CityModel>
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
      @foreach (var city in Model) {
      <tr>
        <td>@city.CityName</td>
        <td>@city.StateName</td>
        <td>@city.CountryName</td>
        <td>
          <a asp-action="Add" asp-route-CityID="@city.CityID">Edit</a>
          <form
            asp-action="Delete"
            asp-route-CityID="@city.CityID"
            method="post"
          >
            <button type="submit">Delete</button>
          </form>
        </td>
      </tr>
      }
    </tbody>
  </table></Lab14.City.Models.CityModel
>
```

**Add.cshtml**

```html
@model Lab14.City.Models.CityModel

<h2>Add/Edit City</h2>
<form method="post" asp-action="Save">
    <label>Country</label>
    <select asp-for="CountryID" asp-items="@(new SelectList(ViewBag.CountryList, "CountryID", "CountryName"))"></select>

    <label>State</label>
    <select asp-for="StateID" id="StateID"></select>

    <label>City Name</label>
    <input asp-for="CityName" />

    <label>City Code</label>
    <input asp-for="CityCode" />

    <button type="submit">Save</button>
</form>

<script>
    $('#CountryID').change(function () {
        $.post('@Url.Action("GetStatesByCountry")', { CountryID: $(this).val() }, function (data) {
            $('#StateID').empty().append('<option>Select State</option>');
            $.each(data, function (i, state) {
                $('#StateID').append(`<option value="${state.StateID}">${state.StateName}</option>`);
            });
        });
    });
</script>
```

---

### 6. **Run the Application**

- Build and run the application.
- Ensure the Web API project is running for the CRUD operations to work correctly.

---
