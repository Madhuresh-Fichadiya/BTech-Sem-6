## Create Web API Code - Controller's action method
```csharp
[HttpGet]
public async Task<Object> GetAll()
{
  var data = //retrieve data from select all sp


    Dictionary<string, dynamic> result = new Dictionary<string, dynamic>();

    if (data != null)
    {
        result.Add("status", "Success");
        result.Add("data", data);
        return Ok(result);
    }
    else
    {
        result.Add("status", "Failure");
        result.Add("data", "Not Found");
        return NotFound(result);
    }
}
```

## Consume Web API - Controller Action Method
```csharp
public async Task<IActionResult> Index()
{
    var response = await _httpClient.GetAsync("api/Student/GetAll");
    if (response.IsSuccessStatusCode)
    {
       JsonOperation<List<StudentModel>> jsonOperation = new JsonOperation<List<StudentModel>>();

       ApiResultData<List<StudentModel>> apiResultData = await jsonOperation.jsonDeserialization(response);
       return View(apiResultData.Data);
    }
    return View();
}
```
## Model Class
```csharp
public class StudentModel
{
    public int? StudentId { get; set; }

    [Required]
    [StringLength(100)]
    public string Name { get; set; }

    [Required]
    [StringLength(20)]
    public string Enrollment { get; set; }

    [Required]
    public int Semester { get; set; }

    [Required]
    public int CourseID { get; set; }

    [Required]
    public int DepartmentID { get; set; }

    [Newtonsoft.Json.JsonIgnore] // used to ignore fields in json payload while posting to api
    public string? DepartmentName { get; set; }

    [Newtonsoft.Json.JsonIgnore]
    public string? CourseName { get; set; }

}
```
## Common Response Class
```csharp
public class ApiResultData<T> where T : class
{
    public string Status { get; set; }
    public T Data { get; set; }
}
public class JsonOperation<T> where T : class
{
    public async Task<ApiResultData<T>> jsonDeserialization(dynamic response)
    { 
        ApiResultData<T> apiResultData = new ApiResultData<T>();
        var jsonResponse = JsonConvert.DeserializeObject<dynamic>(response.Content.ReadAsStringAsync().Result);
        var jsonData = JsonConvert.SerializeObject(jsonResponse.data);
        apiResultData.Data = JsonConvert.DeserializeObject<T>(jsonData);
        apiResultData.Status = jsonResponse["status"];
        return apiResultData;
    }

    public static string JsonSerialization(T model)
    { 
        return JsonConvert.SerializeObject(model);
    }
}
```
