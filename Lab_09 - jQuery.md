### **Add jQuery CDN in the Layout (_Layout.cshtml)**
If your project uses a shared layout file (`_Layout.cshtml`), add the jQuery CDN link inside the `<head>` or just before the closing `</body>` tag for global availability:

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <title>@ViewData["Title"] - YourAppName</title>
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
</head>
<body>
    @RenderBody()
</body>
</html>
```

---

### **Add jQuery CDN in the Specific View**
If you only want jQuery available in the `City Add/Edit` view, add the CDN link inside the `Scripts` section:

```html
@section Scripts {
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <script>
        $(document).ready(function () {
            // Add your jQuery code here...
            alert("jQuery loaded successfully!");
        });
    </script>
}
```

---
### **Step 1: Apply All jQuery Selectors**
We’ll add functionality using jQuery selectors (`<p>`, `<ul>`, `<li>`, `<a>`, `<tr>`, `<button>`).

#### Add Elements to HTML:
Update your HTML to include these elements if not already present:

```html
<div class="example-section">
    <p>Paragraph Example</p>
    <ul>
        <li>Item 1</li>
        <li>Item 2</li>
        <li>Item 3</li>
    </ul>
    <a href="#">Example Link</a>
    <table>
        <tr>
            <td>Row 1</td>
            <td>Data 1</td>
        </tr>
        <tr>
            <td>Row 2</td>
            <td>Data 2</td>
        </tr>
    </table>
    <button id="exampleButton">Click Me</button>
</div>
```

#### Add jQuery Code:
Below is an example applying jQuery selectors to the above elements:

```javascript
$(document).ready(function () {
    // Selectors
    $("p").css("color", "blue"); // Change paragraph color
    $("ul li").css("font-weight", "bold"); // Bold list items
    $("a").css("text-decoration", "none"); // Remove link underline
    $("tr").hover(function () {
        $(this).css("background-color", "lightgrey"); // Highlight rows
    }, function () {
        $(this).css("background-color", "");
    });
    $("button").click(function () {
        alert("Button Clicked!");
    });
});
```

---

### **Step 2: Apply jQuery Form Events**
Enhance the form with events such as `focus`, `blur`, `change`, and `submit`.

#### Add Events to Form Fields:
Modify the script to capture form events:

```javascript
$(document).ready(function () {
    // Form Events
    $("#CityName").focus(function () {
        $(this).css("background-color", "lightyellow");
    }).blur(function () {
        $(this).css("background-color", "");
    });

    $("#CountryID").change(function () {
        alert("Country selection changed!");
    });

    $("form").submit(function (e) {
        e.preventDefault(); // Prevent default form submission
        alert("Form submitted successfully!");
    });
});
```

---

### **Step 3: Apply Mouse & Keyboard Events**
Add interactive behaviors using mouse and keyboard events.

#### Add Mouse and Keyboard Handlers:
Include events like `click`, `dblclick`, `mouseenter`, `mouseleave`, `keypress`, and `keydown`.

```javascript
$(document).ready(function () {
    // Mouse Events
    $("ul li").mouseenter(function () {
        $(this).css("color", "red");
    }).mouseleave(function () {
        $(this).css("color", "");
    });

    $("button").dblclick(function () {
        $(this).text("Double Clicked!");
    });

    // Keyboard Events
    $("#CityCode").keypress(function (e) {
        console.log("Key pressed: " + e.key);
    });

    $("#CityCode").keydown(function (e) {
        if (e.key === "Enter") {
            alert("Enter key pressed in City Code field!");
        }
    });
});
```

---

### **Step 4: Integration in `City Add/Edit` View**
Combine all functionalities in your `City Add/Edit` view.

```html
@section Scripts {
    <script>
        $(document).ready(function () {
            // Apply all selectors
            $("p").css("color", "blue");
            $("ul li").css("font-weight", "bold");
            $("a").css("text-decoration", "none");
            $("tr").hover(function () {
                $(this).css("background-color", "lightgrey");
            }, function () {
                $(this).css("background-color", "");
            });
            $("button").click(function () {
                alert("Button Clicked!");
            });

            // Form Events
            $("#CityName").focus(function () {
                $(this).css("background-color", "lightyellow");
            }).blur(function () {
                $(this).css("background-color", "");
            });

            $("#CountryID").change(function () {
                alert("Country selection changed!");
            });

            $("form").submit(function (e) {
                e.preventDefault();
                alert("Form submitted successfully!");
            });

            // Mouse & Keyboard Events
            $("ul li").mouseenter(function () {
                $(this).css("color", "red");
            }).mouseleave(function () {
                $(this).css("color", "");
            });

            $("button").dblclick(function () {
                $(this).text("Double Clicked!");
            });

            $("#CityCode").keypress(function (e) {
                console.log("Key pressed: " + e.key);
            });

            $("#CityCode").keydown(function (e) {
                if (e.key === "Enter") {
                    alert("Enter key pressed in City Code field!");
                }
            });
        });
    </script>
}
```

---

### **Next Steps**
- Test each functionality in the browser.
- Add or modify jQuery behaviors based on additional requirements.
