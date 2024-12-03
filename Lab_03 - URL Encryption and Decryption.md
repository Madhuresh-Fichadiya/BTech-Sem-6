### Step 1: Create `UrlEncryptor` Class
Create a new folder named `Helper` and add the `UrlEncryptor` class inside it. The class contains two methods: `Encrypt()` and `Decrypt()`.

```csharp
public static class UrlEncryptor
{
    private static readonly string EncryptionKey = "pjsGLNYrMqU6wny4"; // Change this key

    // Method to encrypt a string
    public static string Encrypt(string text)
    {
        using (var aesAlg = Aes.Create())
        {
            aesAlg.Key = Encoding.UTF8.GetBytes(EncryptionKey);
            aesAlg.IV = new byte[16]; // Initialize the IV to 0s

            var encryptor = aesAlg.CreateEncryptor(aesAlg.Key, aesAlg.IV);

            using (var msEncrypt = new MemoryStream())
            {
                using (var csEncrypt = new CryptoStream(msEncrypt, encryptor, CryptoStreamMode.Write))
                using (var swEncrypt = new StreamWriter(csEncrypt))
                {
                    swEncrypt.Write(text);
                }

                return Convert.ToBase64String(msEncrypt.ToArray());
            }
        }
    }

    // Method to decrypt an encrypted string
    public static string Decrypt(string encryptedText)
    {
        using (var aesAlg = Aes.Create())
        {
            aesAlg.Key = Encoding.UTF8.GetBytes(EncryptionKey);
            aesAlg.IV = new byte[16]; // Initialize the IV to 0s

            var decryptor = aesAlg.CreateDecryptor(aesAlg.Key, aesAlg.IV);

            using (var msDecrypt = new MemoryStream(Convert.FromBase64String(encryptedText)))
            using (var csDecrypt = new CryptoStream(msDecrypt, decryptor, CryptoStreamMode.Read))
            using (var srDecrypt = new StreamReader(csDecrypt))
            {
                return srDecrypt.ReadToEnd();
            }
        }
    }
}
```

### Step 2: Encrypt URL Parameter
When displaying data, you can encrypt URL parameters (e.g., `CityID` or `ProductID`) before including them in a link. This is done by calling `UrlEncryptor.Encrypt()`.

```csharp
<tbody id="cityTable">
    @foreach (DataRow row in Model.Rows)
    {
        <tr>
            <td>@row["CityName"]</td>
            <td>@row["StateName"]</td>
            <td>@row["CountryName"]</td>
            <td class="text-center">
                <!-- Edit Button -->
                <a class="btn btn-outline-success btn-xs" asp-controller="City" asp-action="CityAddEdit" 
                   asp-route-CityID="@UrlEncryptor.Encrypt(row["CityID"].ToString())">
                    <i class="bi bi-pencil-fill"></i>
                </a>
                <!-- Delete Button -->
                <a class="btn btn-outline-danger btn-xs" asp-controller="City" asp-action="Delete" 
                   asp-route-CityID="@UrlEncryptor.Encrypt(row["CityID"].ToString())" 
                   onclick="return confirm('Are you sure you want to delete this city?');">
                    <i class="bi bi-x"></i>
                </a>
            </td>
        </tr>
    }
</tbody>
```

### Step 3: Decrypt URL Parameter
When receiving the `CityID` parameter in your action method, decrypt it using `UrlEncryptor.Decrypt()`.

```csharp
#region Delete
public IActionResult Delete(string CityID)
{
    // Decrypt the CityID
    int decryptedCityID = Convert.ToInt32(UrlEncryptor.Decrypt(CityID.ToString()));

    string connectionstr = _configuration.GetConnectionString("ConnectionString");
    
    using (SqlConnection conn = new SqlConnection(connectionstr))
    {
        conn.Open();
        
        using (SqlCommand sqlCommand = conn.CreateCommand())
        {
            sqlCommand.CommandType = CommandType.StoredProcedure;
            sqlCommand.CommandText = "PR_LOC_City_Delete";
            sqlCommand.Parameters.AddWithValue("@CityID", decryptedCityID);
            sqlCommand.ExecuteNonQuery();
        }
    }
    
    return RedirectToAction("Index");
}
#endregion

  #region Add
  public IActionResult CityAddEdit(string? CityID)
  {
      int? decryptedCityID = null;

      // Decrypt only if CityID is not null or empty
      if (!string.IsNullOrEmpty(CityID))
      {
          string decryptedCityIDString = UrlEncryptor.Decrypt(CityID); // Decrypt the encrypted CityID
          decryptedCityID = int.Parse(decryptedCityIDString); // Convert decrypted string to integer
      }

      LoadCountryList();  
      if (decryptedCityID.HasValue)
      {
          string connectionstr = _configuration.GetConnectionString("ConnectionString");
          DataTable dt = new DataTable();
          using (SqlConnection conn = new SqlConnection(connectionstr))
          {
              conn.Open();
              using (SqlCommand objCmd = conn.CreateCommand())
              {
                  objCmd.CommandType = CommandType.StoredProcedure;
                  objCmd.CommandText = "PR_LOC_City_SelectByPK";
                  objCmd.Parameters.Add("@CityID", SqlDbType.Int).Value = decryptedCityID;

                  using (SqlDataReader objSDR = objCmd.ExecuteReader())
                  {
                      dt.Load(objSDR);
                  }
              }
          }

          if (dt.Rows.Count > 0)
          {
              CityModel model = new CityModel();
              foreach (DataRow dr in dt.Rows)
              {
                  model.CityID = Convert.ToInt32(dr["CityID"]);
                  //model.CityID = Convert.ToInt32(UrlEncryptor.Encrypt(decryptedCityID);
                  model.CityName = dr["CityName"].ToString();
                  model.StateID = Convert.ToInt32(dr["StateID"]);
                  model.CountryID = Convert.ToInt32(dr["CountryID"]);
                  model.CityCode = dr["CityCode"].ToString();
                  ViewBag.StateList = GetStateByCountryID(model.CountryID);
              }
              GetStatesByCountry(model.CountryID);
              return View("CityAddEdit", model);
          }
      }

      return View("CityAddEdit");
  }
  #endregion
```

