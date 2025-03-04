# How to Build Custom Tag Helper
## Step 1: Create a `MailToOwnerTagHelper` class
```csharp
[HtmlTargetElement("mail")]
public class MailToOwnerTagHelper : TagHelper
{
    public string mailAddress { get; set; } // Owner's email
    public override void Process(TagHelperContext context, TagHelperOutput output)
    {
        output.TagName = "a";
        output.Attributes.SetAttribute("href", "mailto:" + mailAddress);
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
<mail mail-address="madhuresh.fichadiya@darshan.ac.in">Contact Us</mail>
```
