# ASP.NET Core Web API | GetAll, GetByID & Delete Operation

## Steps to Create the API Project with Above Operations

### 1. **Create a New Web API Project**

1. Open Visual Studio.
2. Select **Create a new project** > **ASP.NET Core Web API**.
3. Name the project `CoffeeShopWebAPI` and click **Next**.
4. Select **.NET 8.0** as the framework and click **Create**.

---

### 2. **Install Required NuGet Packages**

Install the following packages:

```bash
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
dotnet add package Swashbuckle.AspNetCore
dotnet add package Microsoft.Data.SqlClient
dotnet add package Microsoft.AspNetCore.Mvc.NewtonsoftJson (Version 8.0.0)
```
**OR**

In Visual Studio, Go to **Tools** > **NuGet Package Manager** > **Manage Nuget Packages for Solution**
Click On Browse, Search package name from above mentioned packages and install one by one

---

### 3. **Configure Database Connection**

Open the `appsettings.json` file and add your SQL Server connection string:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=YOUR_SERVER_NAME;Database=YOUR_DATABASE_NAME;Trusted_Connection=True;Encrypt=False;"
  }
}
```

Replace `YOUR_SERVER_NAME` and `YOUR_DATABASE_NAME` with your database details.

---

### 4. **Add the CityModel Class**

Create a folder named `Models` and add the `CityModel.cs` file with the following code:

```csharp
namespace CityAPI.Models
{
    public class CityModel
    {
        public int CityID { get; set; }
        public int StateID { get; set; }
        public int CountryID { get; set; }
        public string CityName { get; set; }
        public string CityCode { get; set; }
    }
}
```

---

### 5. **Add the Data Access Layer**

Create a folder named `Data` and add a `CityRepository.cs` file with the following code:

```csharp
using System.Data;
using System.Data.SqlClient;
using CityAPI.Models;
using Microsoft.Extensions.Configuration;

namespace CityAPI.Data
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
    }
}
```

---

### 6. **Add the CityController**

Create a folder named `Controllers` and add API Controller as `CityController.cs` file with the following code:

```csharp
using CityAPI.Data;
using CityAPI.Models;
using Microsoft.AspNetCore.Mvc;

namespace CityAPI.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class CityController : ControllerBase
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
    }
}
```

---

### 7. **Register Dependencies**

In the `Program.cs` file, add the repository to the dependency injection container:

```csharp
using CityAPI.Data;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddControllers();
```
**builder.Services.AddScoped<CityRepository>()**;
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseAuthorization();
app.MapControllers();
app.Run();
```

---

### 8. **SQL Server Stored Procedures**

#### `PR_LOC_City_SelectAll`

```sql
CREATE PROCEDURE PR_LOC_City_SelectAll
AS
BEGIN
    SELECT CityID, StateID, CountryID, CityName, CityCode
    FROM Cities
END
```

#### `PR_LOC_City_SelectByPK`

```sql
CREATE PROCEDURE PR_LOC_City_SelectByPK
    @CityID INT
AS
BEGIN
    SELECT CityID, StateID, CountryID, CityName, CityCode
    FROM Cities
    WHERE CityID = @CityID
END
```

#### `PR_LOC_City_Delete`

```sql
CREATE PROCEDURE PR_LOC_City_Delete
    @CityID INT
AS
BEGIN
    DELETE FROM Cities
    WHERE CityID = @CityID
END
```

---

### 9. **Run and Test the API**

1. Start the project by running `dotnet run` or using Visual Studio's **Run** button.
2. Test the endpoints using Swagger, Postman, or another API testing tool:
   - `GET /api/City` - Get all cities.
   - `GET /api/City/{id}` - Get city by ID.
   - `DELETE /api/City/{id}` - Delete a city.

---
