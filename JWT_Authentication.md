# How to Implement JWT Token

## Steps to Implementation JWT Token for API project
## Step 1: Add following packages

- Microsoft.AspNetCore.Authentication.JwtBearer
- Microsoft.IdentityModel.Tokens
- System.IdentityModel.Tokens.Jwt

## Step 2: Add Jwt Key, Issuer, Audience in appSetting.json
```csharp
"Jwt": {
  "Key": "ThisIsASecretKeyForJwtTokenGeneration",//replace your key
  "Issuer": "https://localhost:5001",
  "Audience": "https://localhost:5001"
}
```
## Step 3: Add authentication service in Program.cs file
```csharp
// Add authentication services
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = builder.Configuration["Jwt:Issuer"],
            ValidAudience = builder.Configuration["Jwt:Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]))
        };
    });

//Add Authorization Support
builder.Services.AddAuthorization();

// Add Swagger with JWT authentication support
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new OpenApiInfo { Title = "My API", Version = "v1" });

    // Configure JWT Authentication in Swagger
    options.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
    {
        Name = "Authorization",
        Type = SecuritySchemeType.Http,
        Scheme = "Bearer",
        BearerFormat = "JWT",
        In = ParameterLocation.Header,
        Description = "Enter 'Bearer {your JWT token}'"
    });

    options.AddSecurityRequirement(new OpenApiSecurityRequirement
    {
        {
            new OpenApiSecurityScheme
            {
                Reference = new OpenApiReference
                {
                    Type = ReferenceType.SecurityScheme,
                    Id = "Bearer"
                }
            },
            new string[] {}
        }
    });
});
```
Do write following after var app = builder.Build(); method
```csharp
// Add middleware for Authentication and Authorization
app.UseAuthentication();
app.UseAuthorization();

// Enable Swagger
app.UseSwagger();
app.UseSwaggerUI(options =>
{
    options.SwaggerEndpoint("/swagger/v1/swagger.json", "My API v1");
});
```

## Step 4: Model Class
```csharp
public class UserModel
{
    public string Username { get; set; }
    public string Password { get; set; }
}
```

## Step 5: Add Authorization Controller
```csharp
[Route("api/[controller]")]
[ApiController]
public class AuthController : ControllerBase
{
    private readonly IConfiguration _config;

    public AuthController(IConfiguration config)
    {
        _config = config;
    }

    [HttpPost("login")]
    public IActionResult Login([FromBody] UserModel user)
    {
        if (user.Username == "admin" && user.Password == "password")
        {
            Dictionary<string, Object> result = new Dictionary<string, Object>();
            var token = GenerateJwtToken(user.Username);
            result.Add("token", token);
            //result.Add("user", user); // you can call SP and get required data then add that object in result.
            return Ok(result);
        }

        return Unauthorized();
    }

    private string GenerateJwtToken(string username) // Change this with UserID
    {
        var securityKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_config["Jwt:Key"]));
        var credentials = new SigningCredentials(securityKey, SecurityAlgorithms.HmacSha256);

        var claims = new[]
        {
            new Claim(JwtRegisteredClaimNames.Sub, username),
            new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString())
        };

        var token = new JwtSecurityToken(
            issuer: _config["Jwt:Issuer"],
            audience: _config["Jwt:Audience"],
            claims: claims,
            expires: DateTime.UtcNow.AddHours(1),
            signingCredentials: credentials
        );

        return new JwtSecurityTokenHandler().WriteToken(token);
    }
}
```
## Step 6: Create one endpoint to test our token

```csharp
[Authorize]
[Route("api/[controller]")]
[ApiController]
public class HomeController : ControllerBase
{
    List<Student> students = new List<Student>()
    {
        new Student(){ StudentID =1, Name="Raj"},
        new Student(){ StudentID =2, Name="Pankaj"},
        new Student(){ StudentID =3, Name="Jay"},
    };
    [HttpGet]
    public List<Student> Index()
    {
        return students;
    }

}

public class Student
{
    public int StudentID { get; set; }
    public string Name { get; set; }    
}
```

# Steps for consume project code
## Step 1: Add Authentication service as follows
```csharp
public class AuthService
{
    private readonly HttpClient _httpClient;

    public AuthService(HttpClient httpClient)
    {
        _httpClient = httpClient;
    }

    public async Task<string?> AuthenticateUserAsync(string username, string password)
    {
        var requestData = new { Username = username, Password = password };
        var content = new StringContent(JsonConvert.SerializeObject(requestData), Encoding.UTF8, "application/json");

        HttpResponseMessage response = await _httpClient.PostAsync("https://localhost:7228/api/Auth/login", content);//replace your endpoint

        if (response.IsSuccessStatusCode)
        {
            var result = await response.Content.ReadAsStringAsync();
            return result; // JWT Token
        }

        return null;
    }
}
```

## Step 2: Add Authentication Controller
```csharp
public class AuthController : Controller
{
    private readonly AuthService _authService;
    private readonly IHttpContextAccessor _httpContextAccessor;

    public AuthController(AuthService authService, IHttpContextAccessor httpContextAccessor)
    {
        _authService = authService;
        _httpContextAccessor = httpContextAccessor;
    }

    [HttpGet]
    public IActionResult Login()
    {
        return View();
    }

    [HttpPost]
    public async Task<IActionResult> Login(string Username, string Password)
    {
        var token = await _authService.AuthenticateUserAsync(Username, Password);

        if (token == null)
        {
            ViewBag.Error = "Invalid credentials.";
            return View();
        }

        _httpContextAccessor.HttpContext.Session.SetString("JWTToken", token);
        return RedirectToAction("Dashboard");
    }

    public IActionResult Dashboard()
    {
        var token = _httpContextAccessor.HttpContext.Session.GetString("JWTToken");
        ViewBag.Token = token;
        return View();
    }

    public IActionResult Logout()
    {
        _httpContextAccessor.HttpContext.Session.Clear();
        return RedirectToAction("Login");
    }
}
```
## Step 3: Login page design
```csharp
@model UserModel;
@{
    ViewData["Title"] = "Login";
}

<h2>Login</h2>

@if (ViewBag.Error != null)
{
    <div class="alert alert-danger">@ViewBag.Error</div>
}

<form method="post" class="col-4">
    <div class="form-group">
        <label>Username</label>
        <input type="text" class="form-control" name="Username" required />
    </div>
    <div class="form-group">
        <label>Password</label>
        <input type="password" class="form-control" name="Password" required />
    </div><br/>
    <div class="form-group">
        <button type="submit" class="btn btn-primary">Login</button>
    </div>
</form>
```
