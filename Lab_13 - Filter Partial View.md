# Filter Partial View
## Step 1: Design a List page
- Here we have taken State list page for filter.
```csharp
<h3>State List</h3>
<hr />
<div class="card">
    <div class="card-body">
        <partial name="_StateListFilter" />
    </div>
</div>
```
Next design Partial view

```csharp
<h5 class="card-title">Apply Filter</h5>

<!-- General Form Elements -->
<form method="post" asp-controller="Home" asp-action="Index">
  <div class="row">
      <div class="col-4 mb-4">
          <label class="col-form-label">Select Country</label>
          <div class="col-10">
              <select id="ddlCountryID" class="form-control" name="CountryID" asp-items="@(new SelectList(ViewBag.CountryList,"CountryID","CountryName"))">
                  <option>Select Country</option>
              </select>
          </div>
      </div>
      <div class="col-4 mb-4">
          <label for="inputText" class="col-4 col-form-label">State Name</label>
          <div class="col-10">
              <input id="txtStateName" type="text" name="StateName" class="form-control" placeholder="Enter State Name">
          </div>
      </div>
      <div class="col-4 mb-4">
          <label for="inputText" class="col-4 col-form-label">State Code</label>
          <div class="col-10">
              <input id="txtStateCode" type="text" name="StateCode" class="form-control" placeholder="Enter State Code">
          </div>
      </div>
  </div>
  <div class="row mb-3">
      <div class="col-sm-10">
          <button type="submit" class="btn btn-primary">Search</button>
          <button class="btn btn-danger" onclick="fnClearControls()" asp-controller="Home" asp-action="ResetForm">Clear</button>
      </div>
  </div>
</form>
```
