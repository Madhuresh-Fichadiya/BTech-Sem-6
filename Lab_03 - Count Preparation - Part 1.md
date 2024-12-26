# Count Preparation - Part 1

## Overview
This guide provides instructions for modifying the database and API to include the count of cities in each state. The changes include updating a stored procedure, adding a new field to the StateModel, and updating the StateRepository for API integration.

---

## Step 1: Alter the Stored Procedure
Update the stored procedure to include the State count for each state.

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
        COUNT([dbo].[City].[StateID]) AS CityCount
    FROM
        [dbo].[State]
    LEFT OUTER JOIN
        [dbo].[Country] ON [dbo].[Country].[CountryID] = [dbo].[State].[CountryID]
    LEFT OUTER JOIN
        [dbo].[City] ON [dbo].[State].[StateID] = [dbo].[City].[StateID]
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

## Step 2: Update the StateModel
Add a new field StateCount to the StateModel in the API.

### Code Changes:
*StateModel.cs*
```csharp
public class StateModel
{
    public int StateID { get; set; }
    public string StateName { get; set; }
    public string StateCode { get; set; }
    public int CountryID { get; set; }
    public DateTime CreatedDate { get; set; }
    public DateTime ModifiedDate { get; set; }
    public int CityCount { get; set; } // New field
}
```

---

## Step 3: Update the StateRepository
Modify the repository to include the StateCount field in the database query result mapping.

### Code Changes:
*StateRepository.cs*
```csharp
public List<StateModel> GetStates()
{
    List<StateModel> states = new List<StateModel>();

    using (SqlCommand cmd = new SqlCommand("PR_LOC_State_SelectAll", connection))
    {
        cmd.CommandType = CommandType.StoredProcedure;
        connection.Open();

        using (SqlDataReader reader = cmd.ExecuteReader())
        {
            while (reader.Read())
            {
                StateModel State = new StateModel
                {
                    StateID = reader.GetInt32(reader.GetOrdinal("StateID")),
                    StateName = reader.GetString(reader.GetOrdinal("StateName")),
                    StateCode = reader.GetString(reader.GetOrdinal("StateCode")),
                    CreatedDate = reader.GetDateTime(reader.GetOrdinal("CreatedDate")),
                    ModifiedDate = reader.GetDateTime(reader.GetOrdinal("ModifiedDate")),
                    CityCount = reader.GetInt32(reader.GetOrdinal("CityCount")) // Mapping StateCount
                };
                states.Add(State);
            }
        }
    }

    return cities;
}
```

---

## Step 4: Add StateCount Field in ConsumeAPI State Model
Update the ConsumeAPI State Model to include the StateCount field.

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

## Step 5: Add StateCount Field in View Page
Update the view page to display the StateCount field in the table.

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
        @foreach (var State in Model)
        {
            <tr>
                <td>@State.StateID</td>
                <td>@State.StateName</td>
                <td>@State.StateCode</td>
                <td>@State.CountryID</td>
                <td>@State.CountryName</td>
                <td>@State.CityCount</td>
            </tr>
        }
    </tbody>
</table>
```

---

## Testing the Changes
1. Run the updated stored procedure in SQL Server to verify the results include the StateCount column.
2. Test the API endpoint that retrieves cities and ensure the StateCount is correctly populated in the response.
3. Verify that the changes work with the frontend or any consumer of the API.

---

## Notes
- Ensure that the database connection string in the application configuration is correct.
- If using Entity Framework, update the corresponding database context and models to reflect these changes.
- Test thoroughly to ensure no breaking changes in existing functionality.
