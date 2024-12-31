# Fluent Validation

This lab demonstrates how to use FluentValidation in an ASP.NET Core Web API project to implement model validation.

---

### 1. Install Required Packages

Run the following commands to install necessary packages:

```bash
dotnet add package FluentValidation.AspNetCore
```

---

### 2. Update `Program.cs`

In the `Program.cs` file, add the following configuration to register FluentValidation:

#### a. Register All Validators in the Assembly:

```csharp
using System.Reflection;

builder.Services.AddControllers()
    .AddFluentValidation(fv =>
        fv.RegisterValidatorsFromAssemblies(new[] { Assembly.GetExecutingAssembly() }));
```

#### b. Register a Single Validator:

If you want to register a single validator instead of all validators in the assembly, use:

```csharp
builder.Services.AddControllers()
    .AddFluentValidation(fv =>
        fv.RegisterValidatorsFromAssemblyContaining<CountryValidator>());
```

---

### 3. Request and Validation Flow

#### Request Flow:

1. The client sends an HTTP request to the API controller.
2. The controller invokes the appropriate method in the `CountryRepository`.
3. The repository interacts with the database using stored procedures.
4. Results are returned to the controller and then to the client.

#### Validation Flow:

1. FluentValidation validates the model when a request hits the controller.
2. If validation passes, the controller calls the repository methods.
3. If validation fails, an error response is returned to the client.

---

### 4. Define the `CountryModel`

Create a `Models` folder and add the `CountryModel` class:

```csharp
namespace api.Model
{
    public class CountryModel
    {
        public int? CountryID { get; set; }
        public string CountryName { get; set; }
        public string CountryCode { get; set; }
    }
}
```

---

### 5. Implement Fluent Validation

Create a `Validators` folder and add the `CountryValidator` class:

```csharp
using api.Model;
using FluentValidation;

namespace api.Validators
{
    public class CountryValidator : AbstractValidator<CountryModel>
    {
        public CountryValidator()
        {
            RuleFor(c => c.CountryName)
                 .NotNull().WithMessage("Country name must not be empty.")
                 .Length(3, 20).WithMessage("Country name must be between 3 and 20 characters.")
                 .Matches("^[A-Za-z ]*$").WithMessage("Country name must contain only letters and spaces."); // Ensures only letters and spaces are allowed

            RuleFor(c => c.CountryCode)
                .NotNull().WithMessage("Country code must not be empty.")
                .MaximumLength(4).WithMessage("Country code must not exceed 4 characters.")
                .Matches("^[A-Za-z]{2,4}$").WithMessage("Country code must contain only letters and be between 2 to 4 characters.");

        }
    }
}

```

---

### 6. Create Repository for Database Operations

Add the `CountryRepository` class for database interactions using stored procedures:

```csharp
using api.Model;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Data.SqlClient;
using System.Data;

namespace api.Data
{
    public class CountryRepository
    {
        private readonly string _connectionString;
        public CountryRepository(IConfiguration configuration)
        {
            _connectionString = configuration.GetConnectionString("ConnectionString");
        }

        #region SelectAll
        public IEnumerable<CountryModel> SelectAllCountry()
        {
            var countries = new List<CountryModel>();
            using (SqlConnection conn = new SqlConnection(_connectionString))
            {
                SqlCommand cmd = new SqlCommand("PR_Country_SelectAll", conn)
                {
                    CommandType = CommandType.StoredProcedure
                };
                conn.Open();
                SqlDataReader reader = cmd.ExecuteReader();
                while (reader.Read())
                {
                    countries.Add(new CountryModel
                    {
                        CountryID = Convert.ToInt32(reader["CountryID"]),
                        CountryName = reader["CountryName"].ToString(),
                        CountryCode = reader["CountryCode"].ToString()
                    });
                }
            }
            return countries;
        }
        #endregion

        #region SelectByID
        public CountryModel SelectCountryByID(int CountryID)
        {
            CountryModel country = null;
            using (SqlConnection conn = new SqlConnection(_connectionString))
            {
                SqlCommand cmd = new SqlCommand("PR_Country_SelectByID", conn)
                {
                    CommandType = CommandType.StoredProcedure
                };

                cmd.Parameters.AddWithValue("@CountryID", CountryID);

                conn.Open();
                SqlDataReader reader = cmd.ExecuteReader();
                while (reader.Read())
                {
                    country = new CountryModel
                    {
                        CountryID = Convert.ToInt32(reader["CountryID"]),
                        CountryName = reader["CountryName"].ToString(),
                        CountryCode = reader["CountryCode"].ToString()
                    };
                }
            }
            return country;
        }
        #endregion

        #region Delete
        public bool DeleteCountry(int CountryID)
        {
            using (SqlConnection conn = new SqlConnection(_connectionString))
            {
                SqlCommand cmd = new SqlCommand("PR_Country_Delete", conn)
                {
                    CommandType = CommandType.StoredProcedure
                };

                cmd.Parameters.AddWithValue("@CountryID", CountryID);
                conn.Open();
                int rowSelected = cmd.ExecuteNonQuery();
                return rowSelected > 0;
            }
        }
        #endregion

        #region Insert
        public bool InsertCountry(CountryModel country)
        {
            using (SqlConnection conn = new SqlConnection(_connectionString))
            {
                SqlCommand cmd = new SqlCommand("PR_Country_Insert", conn)
                {
                    CommandType = CommandType.StoredProcedure
                };
                cmd.Parameters.AddWithValue("@CountryName", country.CountryName);
                cmd.Parameters.AddWithValue("@CountryCode", country.CountryCode);

                conn.Open();
                int rowAffected = cmd.ExecuteNonQuery();

                return rowAffected > 0;
            }
        }
        #endregion

        #region Update
        public bool UpdateCountry(CountryModel country)
        {
            using (SqlConnection conn = new SqlConnection(_connectionString))
            {
                SqlCommand cmd = new SqlCommand("PR_Country_Update", conn)
                {
                    CommandType = CommandType.StoredProcedure
                };
                cmd.Parameters.AddWithValue("@CountryID", country.CountryID);
                cmd.Parameters.AddWithValue("@CountryName", country.CountryName);
                cmd.Parameters.AddWithValue("@CountryCode", country.CountryCode);

                conn.Open();
                int rowAffected = cmd.ExecuteNonQuery();
                return rowAffected > 0;
            }
        }
        #endregion
    }
}

```

---

### 7. Create the Controller

Add the `CountryController` class to handle HTTP requests:

```csharp
using api.Data;
using api.Model;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;

namespace api.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class CountryController : ControllerBase
    {
        private readonly CountryRepository _countryRepository;
        public CountryController(CountryRepository countryRepository)
        {
            _countryRepository = countryRepository;
        }

        [HttpGet]
        public IActionResult GetAllCountry()
        {
            var countryModels = _countryRepository.SelectAllCountry();
            if (countryModels == null)
            {
                return NoContent();
            }
            return Ok(countryModels);
        }

        [HttpGet("{countryID}")]
        public IActionResult GetCountryByID(int countryID)
        {
            var country = _countryRepository.SelectCountryByID(countryID);
            if(country == null)
            {
                return NotFound();
            }
            return Ok(country);
        }

        [HttpDelete("{id}")]
        public IActionResult Delete(int id)
        {
            var isDeleted = _countryRepository.DeleteCountry(id);
            if (!isDeleted)
            {
                return NotFound();
            }
            return NoContent();
        }

        [HttpPost]
        public IActionResult Insert([FromBody] CountryModel country)
        {
            if (country == null)
            {
                return BadRequest();
            }
            bool isInserted = _countryRepository.InsertCountry(country);
            if (isInserted)
            {
                return Ok(new { Message = "Country inserted successfully!" });
            }
            return StatusCode(500, "An error occured while inserting the country.");
        }

        [HttpPut("{CountryID}")]
        public IActionResult Update(int CountryID, [FromBody] CountryModel country)
        {
            if (country == null || CountryID != country.CountryID)
            {
                return BadRequest();
            }
            var isUpdated = _countryRepository.UpdateCountry(country);
            if (!isUpdated)
            {
                return NotFound();
            }
            return NoContent();
        }
    }
}

```


---
