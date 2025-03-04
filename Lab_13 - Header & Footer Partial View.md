#
## Step 1: Prepere header and footer page partial view
- design this page as per requirement, here only sample given
- First design Header partial view
```csharp
<nav class="navbar navbar-expand-sm navbar-toggleable-sm navbar-light bg-white border-bottom box-shadow mb-3">
    <div class="container-fluid">
        <a class="navbar-brand" asp-area="" asp-controller="Home" asp-action="Index"><img class="img img-responsive" width="120px" src="~/Images/Demo.png"></a>
        <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target=".navbar-collapse" aria-controls="navbarSupportedContent"
                aria-expanded="false" aria-label="Toggle navigation">
            <span class="navbar-toggler-icon"></span>
        </button>
        <div class="navbar-collapse collapse d-sm-inline-flex justify-content-between">
            <ul class="navbar-nav flex-grow-1">
                <li class="nav-item">
                    <a class="nav-link text-dark" asp-area="" asp-controller="Home" asp-action="Index">Home</a>
                </li>
                <li class="nav-item">
                    <a class="nav-link text-dark" asp-area="" asp-controller="Home" asp-action="About">About</a>
                </li>
                <li class="nav-item">
                    <a class="nav-link text-dark" asp-area="" asp-controller="Home" asp-action="Contact">Contact</a>
                </li>
            </ul>
        </div>
    </div>
</nav>
```
- Same way design footer partial view
```csharp
<div class="container">
    &copy; 2024 - Darshan University - <a asp-area="" asp-controller="Home" asp-action="Privacy">Privacy</a>
</div>
```
## Step 2: Prepere your project's Layout page
- prepare layout page design as required and render header and footer partial views
```csharp
<header>
    <partial name="_Header" />
</header>
<div class="container">
    <main role="main" class="pb-4">
        @RenderBody()
    </main>
</div>
<div class="pt-2">
<footer class="border-top footer text-muted">
  <partial name="_Footer" />
</footer>
```
