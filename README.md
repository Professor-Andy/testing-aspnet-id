# testing-aspnet-id

## step 1 
create mvc project with identity 
- dotnet new mvc --auth Individual 

## step 2

update Program.cs with the following 


```csharp
builder.Services.AddIdentity<MyUser, IdentityRole>(options => 
    {
        options.SignIn.RequireConfirmedAccount = true; // Keep your original requirement if desired
        // Add other password/lockout options here
    })
    .AddEntityFrameworkStores<ApplicationDbContext>()
    .AddDefaultTokenProviders();

builder.Services.AddAuthorization();
builder.Services.AddRazorPages();
builder.Services.AddControllersWithViews();
```

## step 3 
update AppDbConstext to 
```csharp
public class ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) : IdentityDbContext<MyUser>(options)
{
}
```

## step 4
add the appropriate models 

```csharp 

public class MyUser : IdentityUser
{
    public string? FirstName { get; set; }
    public string? LastName { get; set; }
    public string? Address { get; set; }
    public string? City { get; set; }
    public string? State { get; set; }
    public string? ZipCode { get; set; }
    public string? Country { get; set; } 
}
namespace testing_aspnet_id.Models
{
    public class RegisterViewModel
    {
        [Required]
        [EmailAddress]
        public string? Email { get; set; }

        [Required]
        [DataType(DataType.Password)]
        public string? Password { get; set; }

        [DataType(DataType.Password)]
        [Compare("Password", ErrorMessage = "The password and confirmation password do not match.")]
        public string? ConfirmPassword { get; set; }

        [Required]
        public string? FirstName { get; set; }

        [Required]
        public string? LastName { get; set; }
        
        public string? Address { get; set; }
        public string? City { get; set; }
        public string? State { get; set; }
        public string? ZipCode { get; set; }
        public string? Country { get; set; }
        public string? PhoneNumber { get; set; }
    }
}
```

## step 5
create a registration view 
```razor
@model RegisterViewModel

@{
    ViewData["Title"] = "Register";
}

<h2>Register</h2>

<form asp-action="Register" method="post">
    <div asp-validation-summary="All" class="text-danger"></div>

    <div class="form-group">
        <label asp-for="Email"></label>
        <input asp-for="Email" class="form-control" />
        <span asp-validation-for="Email" class="text-danger"></span>
    </div>

    <div class="form-group">
        <label asp-for="Password"></label>
        <input asp-for="Password" class="form-control" />
        <span asp-validation-for="Password" class="text-danger"></span>
    </div>

    <div class="form-group">
        <label asp-for="ConfirmPassword"></label>
        <input asp-for="ConfirmPassword" class="form-control" />
        <span asp-validation-for="ConfirmPassword" class="text-danger"></span>
    </div>

    <div class="form-group">
        <label asp-for="FirstName"></label>
        <input asp-for="FirstName" class="form-control" />
    </div>

    <div class="form-group">
        <label asp-for="LastName"></label>
        <input asp-for="LastName" class="form-control" />
    </div>

    <div class="form-group">
        <label asp-for="Address"></label>
        <input asp-for="Address" class="form-control" />
    </div>

    <div class="form-row">
        <div class="form-group col-md-6">
            <label asp-for="City"></label>
            <input asp-for="City" class="form-control" />
        </div>
        <div class="form-group col-md-4">
            <label asp-for="State"></label>
            <input asp-for="State" class="form-control" />
        </div>
        <div class="form-group col-md-2">
            <label asp-for="ZipCode"></label>
            <input asp-for="ZipCode" class="form-control" />
        </div>
    </div>

    <div class="form-group">
        <label asp-for="Country"></label>
        <input asp-for="Country" class="form-control" />
    </div>

    <div class="form-group">
        <label asp-for="PhoneNumber"></label>
        <input asp-for="PhoneNumber" class="form-control" />
    </div>

    <button type="submit" class="btn btn-primary mt-3">Register</button>
</form>

@section Scripts {
    @{await Html.RenderPartialAsync("_ValidationScriptsPartial");}
}
``` 

## step 6
add dependency injection 
```csharp 
    private readonly UserManager<MyUser> _userManager;
    private readonly SignInManager<MyUser> _signInManager;

    public HomeController(UserManager<MyUser> userManager, SignInManager<MyUser> signInManager)
    {
        _userManager = userManager;
        _signInManager = signInManager;
    }
```

## step 7
update the home controller 

```csharp
    [HttpGet]
    public IActionResult Register()
    {
        return View();
    }

    [HttpPost]
    [ValidateAntiForgeryToken]
    public async Task<IActionResult> Register(RegisterViewModel model)
    {
        if (ModelState.IsValid)
        {
            if (model.Password == null)
        {
            ModelState.AddModelError(string.Empty, "Password is required.");
            return View(model);
        }
            var user = new MyUser
            {
                UserName = model.Email,
                Email = model.Email,
                FirstName = model.FirstName,
                LastName = model.LastName,
                Address = model.Address,
                City = model.City,
                State = model.State,
                ZipCode = model.ZipCode,
                Country = model.Country,
                PhoneNumber = model.PhoneNumber
            };

            var result = await _userManager.CreateAsync(user, model.Password);

            if (result.Succeeded)
            {
                // Optionally assign default roles here if configured
                await _signInManager.SignInAsync(user, isPersistent: false);
                return RedirectToAction("Index", "Home");
            }

            foreach (var error in result.Errors)
            {
                ModelState.AddModelError(string.Empty, error.Description);
            }
        }

        // If we got this far, something failed; redisplay form
        return View(model);
    }
```
## step 8
install and migrate changes to db

```bash 
dotnet tool install dotnet-ef
dotnet dotnet-ef migrations add InitialCreate
dotnet dotnet-ef database update
```

## step 9

```bash
dotnet run 
```

## other login options

```csharp
builder.Services.ConfigureApplicationCookie(options =>
{
    // The lifetime of the authentication cookie (e.g., user is signed out after 30 minutes of total time)
    options.ExpireTimeSpan = TimeSpan.FromMinutes(30);

    // Enables sliding expiration: if the user is active, the cookie lifetime is reset halfway through
    options.SlidingExpiration = true;

    // Optional: Custom login path if a user's session times out and they need to re-authenticate
    options.LoginPath = "/Account/Login";
    
    // Optional: Access denied path
    options.AccessDeniedPath = "/Account/AccessDenied";

    options.SignIn.RequireConfirmedAccount = true;     // Requires email confirmation before sign-in
    options.SignIn.RequireConfirmedEmail = false;       // Requires confirmed email address specifically
    options.SignIn.RequireConfirmedPhoneNumber = false; // Requires confirmed phone number

    options.Password.RequiredLength = 8;           // Minimum password length
    options.Password.RequiredUniqueChars = 1;      // Minimum number of unique characters
    options.Password.RequireDigit = true;          // Requires 0-9
    options.Password.RequireLowercase = true;      // Requires a-z
    options.Password.RequireUppercase = true;      // Requires A-Z
    options.Password.RequireNonAlphanumeric = true;// Requires special character (e.g. !@#)

    options.Lockout.AllowedForNewUsers = true;     // Enables lockout for newly created accounts
    options.Lockout.DefaultLockoutTimeSpan = TimeSpan.FromMinutes(5); // Duration of lockout after max failures
    options.Lockout.MaxFailedAccessAttempts = 5;   // Number of failed attempts before lockout triggers
});
```

