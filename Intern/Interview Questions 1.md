### Notes on ASP.NET Core Interview Questions

#### 1. **What is .NET Core?**
   - A free and open-source framework developed by Microsoft for building applications that can run on multiple operating systems (Windows, Linux, Mac).

#### 2. **Advantages of .NET Core over .NET Framework:**
   - **Cross-Platform:** Runs on multiple OS.
   - **Open Source:** Free to use and modify.
   - **Integration with Modern UI Frameworks:** Easier integration with Angular, React, etc.
   - **Hosting Flexibility:** Can be hosted on various servers.
   - **Built-in Dependency Injection:** Promotes loosely coupled design.
   - **Multiple IDE Support:** Works with Visual Studio, Visual Studio for Mac, and Visual Studio Code.

#### 3. **Default Project Structure in ASP.NET Core:**
   - **wwwroot:** Stores static files (JS, CSS, images).
   - **Program.cs:** Entry point of the application.
   - **Startup.cs:** Configures services and request pipeline.
   - **appsettings.json:** Stores configuration settings (e.g., database connection strings).

#### 4. **Role of Program.cs:**
   - Entry point of the application; builds the host server and runs the application.

#### 5. **Role of Startup.cs:**
   - Configures services and the request pipeline. Contains methods like `ConfigureServices` and `Configure`.

#### 6. **Dependency Injection:**
   - Promotes loosely coupled design by allowing services to be injected into controllers rather than instantiated directly.

#### 7. **Service Lifetimes:**
   - **Singleton:** One instance shared across all requests.
   - **Scoped:** One instance per request.
   - **Transient:** A new instance is created for every request.

#### 8. **Middleware:**
   - Components executed on every request, configured in the `Startup.cs` file.

#### 9. **Hosting Models:**
   - **In-Process Hosting:** Uses IIS directly.
   - **Out-of-Process Hosting:** Uses Kestrel as a web server, with IIS as a reverse proxy.

#### 10. **Kestrel vs. IIS:**
   - Kestrel is lightweight and cross-platform; IIS is a full-featured web server with more complex capabilities.

#### 11. **Request Delegates and Middleware Methods:**
   - **Use:** Executes the next middleware.
   - **Run:** Terminates the middleware pipeline.
   - **Map:** Executes middleware based on URL patterns.

#### 12. **Configuration Settings:**
   - Stored in `appsettings.json`, Azure Key Vault, environment variables, etc.

#### 13. **Routing:**
   - Handles incoming HTTP requests based on URL patterns.

#### 14. **Attribute Routing:**
   - Allows manipulation of URL behavior using route attributes.

#### 15. **Request Processing Pipeline:**
   - Flow: Middleware → Routing → Controller Initialization → Action Execution → View Result → Response.

#### 16. **Compatibility with .NET Framework:**
   - ASP.NET Core applications can work with .NET Framework 4.x.

#### 17. **Meta Packages:**
   - Packages that include multiple dependencies for easier management.

#### 18. **Static Files:**
   - Served from the `wwwroot` folder using `app.UseStaticFiles()`.

#### 19. **JSON Files in ASP.NET Core:**
   - **global.json:** Solution-level settings.
   - **launchSettings.json:** Environment variables.
   - **appsettings.json:** Configuration settings.
   - **bundleConfig.js:** Bundling and minification settings.
   - **project.json:** Project-level configuration.

#### 20. **.NET Standard:**
   - A set of APIs that frameworks must implement to be compliant.

#### 21. **Razor Pages:**
   - A page-centric development model in ASP.NET Core.

#### 22. **Dependency Injection in Views:**
   - Use the `@inject` directive to inject services into views.

#### 23. **Custom Middleware:**
   - Create a custom middleware class and add it to the pipeline in `Startup.cs`.

#### 24. **Session and State Management:**
   - Techniques include cookies, sessions, temp data, query strings, and hidden fields.

#### 25. **Model Validation:**
   - Achieved using data annotations to enforce validation rules.

#### 26. **Error Handling:**
   - Configured in `Startup.cs` to display detailed errors in development and custom errors in production.

#### 27. **Enabling Session:**
   - Use `AddSession()` in `ConfigureServices` and `app.UseSession()` in `Configure`.

#### 28. **Model Binding:**
   - Maps data from HTTP requests to action method parameters automatically.

These notes summarize key concepts and questions that may arise in ASP.NET Core interviews, providing a solid foundation for preparation.