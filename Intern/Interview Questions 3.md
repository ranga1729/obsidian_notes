Here are some commonly asked **ASP.NET MVC interview questions** for an **intern .NET developer position**, along with their **answers**:

---

## **1. Basic Questions and Answers**

### **1. What is MVC in ASP.NET?**

**Answer:**  
MVC (Model-View-Controller) is a design pattern used for developing web applications. It separates the application logic into three components:

- **Model**: Represents the data and business logic.
- **View**: Handles UI/Presentation logic.
- **Controller**: Manages user input, interacts with the model, and renders views.

---

### **2. Explain the different components of MVC.**

**Answer:**  
The three main components of MVC are:

- **Model**: Manages the data and business logic of the application.
- **View**: Responsible for rendering the UI elements.
- **Controller**: Handles user requests, processes data through the Model, and returns a response via the View.

---

### **3. What is the difference between MVC and WebForms?**

**Answer:**

|Feature|ASP.NET WebForms|ASP.NET MVC|
|---|---|---|
|Architecture|Follows event-driven model|Follows MVC pattern|
|ViewState|Uses ViewState to maintain state|No ViewState (stateless)|
|Page Lifecycle|Complex|Simple|
|Testability|Difficult to test|Easy to test|
|HTML Control|Uses server controls|Uses HTML helpers|

---

### **4. What is Routing in MVC?**

**Answer:**  
Routing in MVC is the process of mapping **URLs to Controller actions**. It helps create SEO-friendly URLs and removes the need for physical files.  
Example:

```csharp
public class RouteConfig  
{  
    public static void RegisterRoutes(RouteCollection routes)  
    {  
        routes.MapRoute(  
            name: "Default",  
            url: "{controller}/{action}/{id}",  
            defaults: new { controller = "Home", action = "Index", id = UrlParameter.Optional }  
        );  
    }  
}  
```

---

### **5. What are Action Methods in MVC?**

**Answer:**  
Action methods in MVC are **public methods in a Controller** that handle user requests. Example:

```csharp
public class HomeController : Controller  
{  
    public ActionResult Index()  
    {  
        return View();  
    }  
}
```

---

### **6. What is the difference between ViewData, ViewBag, and TempData?**

**Answer:**

|Feature|ViewData|ViewBag|TempData|
|---|---|---|---|
|Type|Dictionary object|Dynamic property|Dictionary object|
|Scope|Available in the same request|Available in the same request|Available across multiple requests (Redirects)|
|Usage|`ViewData["Message"] = "Hello"`|`ViewBag.Message = "Hello"`|`TempData["Message"] = "Hello"`|

---

### **7. What is a ViewModel in MVC?**

**Answer:**  
A **ViewModel** is a custom class that **combines multiple models** into a single object for the View.  
Example:

```csharp
public class EmployeeViewModel  
{  
    public string Name { get; set; }  
    public string Department { get; set; }  
    public int Salary { get; set; }  
}
```

---

## **2. Intermediate Questions and Answers**

### **8. What is Model Binding in MVC?**

**Answer:**  
Model binding **automatically maps form data to model properties**. Example:

```csharp
public ActionResult Submit(Employee emp)  
{  
    return View(emp);  
}
```

This automatically binds form fields (e.g., Name, Email) to the Employee model.

---

### **9. What are Filters in MVC?**

**Answer:**  
Filters are used to **apply logic before or after an action method executes**. Types of filters:

- **Authorization Filter** – Used for authentication
- **Action Filter** – Runs before/after an action executes
- **Result Filter** – Runs before/after a view result
- **Exception Filter** – Handles errors globally

Example of a custom filter:

```csharp
public class MyCustomFilter : ActionFilterAttribute  
{  
    public override void OnActionExecuting(ActionExecutingContext filterContext)  
    {  
        // Custom logic before action execution  
    }  
}
```

---

### **10. What is Unobtrusive JavaScript in MVC?**

**Answer:**  
Unobtrusive JavaScript means **separating JavaScript from HTML** to keep code cleaner and maintainable.  
Example using **jQuery Validation**:

```html
@Html.TextBoxFor(m => m.Name, new { @class = "form-control" })  
@Html.ValidationMessageFor(m => m.Name)
```

This enables validation without inline JavaScript.

---

### **11. What is the Repository Pattern in MVC?**

**Answer:**  
The **Repository Pattern** is used to **abstract database operations** from the application logic.  
Example:

```csharp
public interface IEmployeeRepository  
{  
    IEnumerable<Employee> GetAllEmployees();  
}  
```

```csharp
public class EmployeeRepository : IEmployeeRepository  
{  
    private ApplicationDbContext _context;  
    public IEnumerable<Employee> GetAllEmployees()  
    {  
        return _context.Employees.ToList();  
    }  
}
```

---

## **3. Advanced Questions and Answers**

### **12. What is Dependency Injection (DI) in MVC?**

**Answer:**  
DI allows for **decoupling dependencies** between classes. Example using **Constructor Injection**:

```csharp
public class EmployeeService  
{  
    private IEmployeeRepository _repo;  
    public EmployeeService(IEmployeeRepository repo)  
    {  
        _repo = repo;  
    }  
}
```

This makes **unit testing easier**.

---

### **13. What is the Anti-Forgery Token in MVC?**

**Answer:**  
It prevents **CSRF (Cross-Site Request Forgery) attacks**.  
Usage in View:

```html
@Html.AntiForgeryToken()
```

Usage in Controller:

```csharp
[HttpPost]  
[ValidateAntiForgeryToken]  
public ActionResult Submit(Employee emp)  
{  
    return View();  
}
```

---

### **14. How do you handle errors in MVC?**

**Answer:**

- **Use `try-catch` blocks**
- **Custom error pages in Web.config**

```xml
<customErrors mode="On">
    <error statusCode="404" redirect="~/Error/PageNotFound"/>
</customErrors>
```

- **Exception Filters (`HandleError` attribute)**

```csharp
[HandleError]
public class HomeController : Controller  
{  
    public ActionResult Index()  
    {  
        throw new Exception("Error occurred!");  
    }  
}
```

---

### **15. How do you implement AJAX in MVC?**

**Answer:**  
Use **jQuery AJAX** to call an MVC action:

```javascript
$.ajax({
    url: "/Employee/GetDetails",
    type: "GET",
    success: function(data) {
        console.log(data);
    }
});
```

MVC Action:

```csharp
public JsonResult GetDetails()  
{  
    var emp = new Employee { Name = "John", Age = 30 };  
    return Json(emp, JsonRequestBehavior.AllowGet);  
}
```

---

## **4. Coding Questions**

1. **Create a Controller with a simple Action method that returns a View.**
2. **Write a LINQ query to fetch data from the database in an MVC application.**
3. **Implement a form submission using POST in MVC.**
4. **Write a custom Action Filter in MVC.**
5. **Use AJAX to fetch and display data in an MVC View.**

---

Would you like me to provide **sample code** for any of these questions? 😊