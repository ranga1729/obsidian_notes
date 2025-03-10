Here are some common .NET interview questions along with their answers:

### 1. **What is .NET?**
   - **Answer:** .NET is a free, cross-platform, open-source developer platform for building many different types of applications. It supports multiple languages, libraries, and frameworks to build web, mobile, desktop, games, and IoT applications.

### 2. **What is the difference between .NET Core and .NET Framework?**
   - **Answer:** 
     - **.NET Core** is a cross-platform, high-performance, open-source framework for building modern, cloud-based, and internet-connected applications. It can run on Windows, macOS, and Linux.
     - **.NET Framework** is a Windows-only framework for building and running applications on Windows. It is not cross-platform and is considered legacy for new development.

### 3. **What is CLR in .NET?**
   - **Answer:** CLR (Common Language Runtime) is the execution engine of .NET. It provides services like memory management, garbage collection, exception handling, and security. CLR converts Intermediate Language (IL) code into native machine code during execution.

### 4. **What is the difference between managed and unmanaged code?**
   - **Answer:**
     - **Managed Code:** Code that is executed by the CLR. It benefits from services like garbage collection, memory management, and exception handling.
     - **Unmanaged Code:** Code that is executed directly by the operating system. It does not benefit from CLR services and is typically written in languages like C or C++.

### 5. **What is Garbage Collection in .NET?**
   - **Answer:** Garbage Collection (GC) is an automatic memory management feature in .NET. It reclaims memory by freeing up objects that are no longer in use, preventing memory leaks and optimizing memory usage.

### 6. **What is the difference between `==` and `.Equals()` in C#?**
   - **Answer:**
     - `==` is a reference equality operator for reference types and a value equality operator for value types.
     - `.Equals()` is a method that can be overridden to provide value equality comparison for both reference and value types.

### 7. **What is the difference between `String` and `StringBuilder` in C#?**
   - **Answer:**
     - **String:** Immutable, meaning once created, it cannot be changed. Any modification results in a new string object.
     - **StringBuilder:** Mutable, meaning it can be modified without creating a new object. It is more efficient for frequent string manipulations.

### 8. **What is an Assembly in .NET?**
   - **Answer:** An assembly is a compiled code library used in .NET applications. It contains Intermediate Language (IL) code, metadata, and resources. Assemblies can be executables (`.exe`) or libraries (`.dll`).

### 9. **What is the difference between `abstract` and `interface` in C#?**
   - **Answer:**
     - **Abstract Class:** Can have both abstract (unimplemented) and concrete (implemented) methods. It can also have fields and constructors.
     - **Interface:** Can only have method signatures (unimplemented methods). It cannot have fields or constructors. A class can implement multiple interfaces but inherit only one abstract class.

### 10. **What is Dependency Injection in .NET?**
   - **Answer:** Dependency Injection (DI) is a design pattern used to achieve Inversion of Control (IoC) between classes and their dependencies. It allows objects to be passed into a class (via constructor, method, or property) rather than the class creating them itself. .NET Core has built-in support for DI.

### 11. **What is the difference between `Task` and `Thread` in C#?**
   - **Answer:**
     - **Thread:** Represents an actual OS-level thread with its own stack and resources. It is lower-level and more resource-intensive.
     - **Task:** Represents an asynchronous operation that may or may not run on a separate thread. It is higher-level and more efficient for asynchronous programming.

### 12. **What is LINQ in .NET?**
   - **Answer:** LINQ (Language Integrated Query) is a feature in .NET that allows querying collections, databases, and XML using a SQL-like syntax directly in C# or VB.NET. It provides a consistent way to query different data sources.

### 13. **What is the difference between `IEnumerable` and `IQueryable` in C#?**
   - **Answer:**
     - **IEnumerable:** Used for in-memory collections. Queries are executed in-memory on the client side.
     - **IQueryable:** Used for querying data from out-of-memory sources like databases. Queries are translated into SQL and executed on the server side.

### 14. **What is Entity Framework in .NET?**
   - **Answer:** Entity Framework (EF) is an Object-Relational Mapping (ORM) framework that enables developers to work with databases using .NET objects. It eliminates the need for most of the data-access code by providing a higher-level abstraction.

### 15. **What is the difference between `async` and `await` in C#?**
   - **Answer:**
     - **async:** A keyword used to define an asynchronous method. It allows the method to use the `await` keyword.
     - **await:** A keyword used to pause the execution of an async method until the awaited task completes. It allows for non-blocking asynchronous operations.

### 16. **What is the Global Assembly Cache (GAC) in .NET?**
   - **Answer:** The GAC is a machine-wide cache for storing shared .NET assemblies. It allows multiple applications to share the same assembly, reducing redundancy and ensuring versioning.

### 17. **What is the difference between `ref` and `out` parameters in C#?**
   - **Answer:**
     - **ref:** Requires the variable to be initialized before being passed to the method. The method can read and modify the value.
     - **out:** Does not require the variable to be initialized before being passed to the method. The method must assign a value before returning.

### 18. **What is a Delegate in C#?**
   - **Answer:** A delegate is a type-safe function pointer that holds a reference to a method. It allows methods to be passed as parameters, returned from other methods, and stored in variables.

### 19. **What is the difference between `throw` and `throw ex` in C#?**
   - **Answer:**
     - **throw:** Preserves the original stack trace of the exception.
     - **throw ex:** Resets the stack trace, making it harder to debug the origin of the exception.

### 20. **What is the purpose of the `using` statement in C#?**
   - **Answer:** The `using` statement is used to ensure that resources (like file handles, database connections, etc.) are properly disposed of when they are no longer needed. It automatically calls the `Dispose` method on the object when the block is exited.