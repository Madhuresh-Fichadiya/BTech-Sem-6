# Create | Web API – [Part – 2] | Insert, Update & DDL Filling Operations

### Steps for Insert, Update, DDL (Use Table City)

## Prerequisites

**Database Setup**

- Ensure the `City` table exists with fields: `CityID (PK, Auto Increment)`, `StateID`, `CountryID`, `CityName`, `CityCode`, `Created`, `Modified`.
- Add stored procedures for data operations:
  - `PR_LOC_City_Insert`
  - `PR_LOC_City_Update`
  - `PR_LOC_Country_SelectComboBox`
  - `PR_LOC_State_SelectComboBoxByCountryID`

**Packages mentioned in Part-1 (previous lab), Model Classes, Database Configuration**

## Steps

### 1. **Create Repository Layer**

#### Insert Method:

Inserts a new city into the database.

```csharp
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
```

#### Update Method:

Updates an existing city in the database.

```csharp
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
```

#### GetCountries Method:

Fetches the list of countries for dropdowns.

```csharp
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
```

#### GetStatesByCountryID Method:

Fetches the list of states by `CountryID` for dropdowns.

```csharp
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
```

---

### 2. **Create Controller Methods**

#### Insert API:

```csharp
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
```

#### Update API:

```csharp
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
```

#### Fetch Countries API:

```csharp
[HttpGet("countries")]
public IActionResult GetCountries()
{
    var countries = _cityRepository.GetCountries();
    if (!countries.Any())
        return NotFound("No countries found.");

    return Ok(countries);
}
```

#### Fetch States by Country ID API:

```csharp
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
```

---

### 3. **Test the APIs**

#### Insert Request:

**POST** `/api/city`  
Body:

```json
{
  "cityID": 0,
  "stateID": 1042,
  "countryID": 1067,
  "cityName": "New City",
  "cityCode": "NC001"
}
```

#### Update Request:

**PUT** `/api/city/{id}`  
Body:

```json
{
  "cityID": 1,
  "stateID": 1042,
  "countryID": 1067,
  "cityName": "Updated City",
  "cityCode": "UC002"
}
```

#### Fetch Countries:

**GET** `/api/city/countries`

#### Fetch States by Country ID:

**GET** `/api/city/states/1067`

---

### 4. **Database Stored Procedures**

#### Insert Procedure:

```sql
CREATE PROCEDURE PR_LOC_City_Insert
    @CityName   VARCHAR(50),
    @CityCode   VARCHAR(50),
    @CountryID  INT,
    @StateID    INT,
    @Created    DATETIME,
    @Modified   DATETIME
AS
BEGIN
    INSERT INTO LOC_City (CityName, CityCode, CountryID, StateID, Created, Modified)
    VALUES (@CityName, @CityCode, @CountryID, @StateID, ISNULL(@Created, GETDATE()), ISNULL(@Modified, GETDATE()));
END;
```

#### Update Procedure:

```sql
CREATE PROCEDURE PR_LOC_City_Update
    @CityID     INT,
    @CityName   VARCHAR(50),
    @CityCode   VARCHAR(50),
    @CountryID  INT,
    @StateID    INT,
    @Modified   DATETIME
AS
BEGIN
    UPDATE LOC_City
    SET CityName = @CityName,
        CityCode = @CityCode,
        CountryID = @CountryID,
        StateID = @StateID,
        Modified = ISNULL(@Modified, GETDATE())
    WHERE CityID = @CityID;
END;
```
