# How to Build Custom Tag Helper
## Step 1: Create a `AlertTagHelper` class
We need to inherit `TagHelper` class to our `AlertTagHelper` class and implement it as follows.
- `[HtmlTargetElement("alert")]` - Specifies that this tag helper will target <alert> elements in Razor views.
- `Type` - Defines a property Type that specifies the type of the alert (e.g., success, warning, info, etc.).
- `Message` - Defines a property Message for the alert's content.
- `Process` - Overrides the Process method, which executes when the Razor engine processes the tag helper.
- `TagHelperContext` - Provides information about the HTML element being processed (e.g., attributes, tag name).
- `TagHelperOutput` - Represents the HTML element being generated or modified by the tag helper.
```csharp
[HtmlTargetElement("alert")]
public class AlertTagHelper : TagHelper
{
    public string Type { get; set; } = "info"; // Default type is 'info'
    public string Message { get; set; } = string.Empty; //This message prop will be used from view page

    public override void Process(TagHelperContext context, TagHelperOutput output)
    {
        output.TagName = "div"; // Generates a <div> element
        output.Attributes.SetAttribute("class", $"alert alert-{Type}");
        output.Content.SetContent(Message);
    }
}
```
## Step 2: Register the namespace of `AlertTagHelper` class in `_ViewImport.cshtml`
```csharp
@addTagHelper *, namespaceOfCustomTagHelper
```

## Step 3: Use the custom tag helper as required.
- We can use the alert tag helper as shown below. also can conditionaly render messages for CRUD operation.
```csharp
<alert message="This is success message" type="success"></alert>
<alert message="This is danger message" type="danger"></alert>
<alert message="This is warning message" type="warning"></alert>
```
