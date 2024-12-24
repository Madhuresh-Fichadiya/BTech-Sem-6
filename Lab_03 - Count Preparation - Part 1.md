# Count Preparation - Part 1

## Overview
This guide provides instructions for modifying the database and API to include the count of cities in each state. The changes include updating a stored procedure, adding a new field to the CityModel, and updating the StateRepository for API integration.

---

## Step 1: Alter the Stored Procedure
Update the stored procedure to include the city count for each state.

```sql
ALTER PROCEDURE [dbo].[PR_LOC_State_SelectAll]
AS
BEGIN
    SELECT
        [dbo].[State].[StateID],
        [dbo].[State].[StateName],
        [dbo].[State].[StateCode],
        [dbo].[Country].[CountryID],
        [dbo].[Country].[CountryName],
        [dbo].[State].[CreatedDate],
        [dbo].[State].[ModifiedDate],
        COUNT([dbo].[City].[CityID]) AS CityCount
    FROM
        [dbo].[State]
    LEFT OUTER JOIN
        [dbo].[Country] ON [dbo].[Country].[CountryID] = [dbo].[State].[CountryID]
    LEFT OUTER JOIN
        [dbo].[City] ON [dbo].[City].[StateID] = [dbo].[State].[StateID]
    GROUP BY
        [dbo].[State].[StateID],
        [dbo].[State].[StateName],
        [dbo].[State].[StateCode],
        [dbo].[Country].[CountryID],
        [dbo].[Country].[CountryName],
        [dbo].[State].[CreatedDate],
        [dbo].[State].[ModifiedDate];
END;
```

---

## Step 2: Update the CityModel
Add a new field CityCount to the CityModel in the API.

### Code Changes:
*CityModel.cs*
```csharp
public class CityModel
{
    public int StateID { get; set; }
    public string CityName { get; set; }
    public string CityCode { get; set; }
    public DateTime CreatedDate { get; set; }
    public DateTime ModifiedDate { get; set; }
    public int CityCount { get; set; } // New field
}
```

---

## Step 3: Update the StateRepository
Modify the repository to include the CityCount field in the database query result mapping.

### Code Changes:
*StateRepository.cs*
```csharp
public List<CityModel> GetCities()
{
    List<CityModel> cities = new List<CityModel>();

    using (SqlCommand cmd = new SqlCommand("PR_LOC_State_SelectAll", connection))
    {
        cmd.CommandType = CommandType.StoredProcedure;
        connection.Open();

        using (SqlDataReader reader = cmd.ExecuteReader())
        {
            while (reader.Read())
            {
                CityModel city = new CityModel
                {
                    StateID = reader.GetInt32(reader.GetOrdinal("StateID")),
                    CityName = reader.GetString(reader.GetOrdinal("CityName")),
                    CityCode = reader.GetString(reader.GetOrdinal("CityCode")),
                    CreatedDate = reader.GetDateTime(reader.GetOrdinal("CreatedDate")),
                    ModifiedDate = reader.GetDateTime(reader.GetOrdinal("ModifiedDate")),
                    CityCount = reader.GetInt32(reader.GetOrdinal("CityCount")) // Mapping CityCount
                };
                cities.Add(city);
            }
        }
    }

    return cities;
}
```

---

## Step 4: Add CityCount Field in ConsumeAPI State Model
Update the ConsumeAPI State Model to include the CityCount field.

### Code Changes:
*StateModel.cs*
```csharp
public class StateModel
{
    public int StateID { get; set; }
    public string StateName { get; set; }
    public string StateCode { get; set; }
    public int CountryID { get; set; }
    public string CountryName { get; set; }
    public DateTime CreatedDate { get; set; }
    public DateTime ModifiedDate { get; set; }
    public int CityCount { get; set; } // New field
}
```

---

## Step 5: Add CityCount Field in View Page
Update the view page to display the CityCount field in the table.

### Code Changes:
*View.cshtml*
```html
<table class="table table-striped">
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
                <td>@city.CityCount</td>
            </tr>
        }
    </tbody>
</table>
```

---

## Testing the Changes
1. Run the updated stored procedure in SQL Server to verify the results include the CityCount column.
2. Test the API endpoint that retrieves cities and ensure the CityCount is correctly populated in the response.
3. Verify that the changes work with the frontend or any consumer of the API.

---

## Notes
- Ensure that the database connection string in the application configuration is correct.
- If using Entity Framework, update the corresponding database context and models to reflect these changes.
- Test thoroughly to ensure no breaking changes in existing functionality.
