# .NET 8 / .NET (C#) Clean Code Practices

This document provides guidelines for writing clean, maintainable, and scalable code in .NET 8 using C#. These practices ensure consistency, readability, and efficiency within the codebase.

---

## **1. General Principles**
1. **Readability Over Obscurity**:
   - Write code that others (and your future self) can understand.
   - Always aim for self-documenting code.
2. **Keep It Simple (KISS)**:
   - Avoid over-engineering solutions. Keep implementations as simple as needed to solve the problem.
3. **Single Responsibility Principle (SRP)**:
   - Every class, method, or function should have only one reason to change.
4. **DRY (Don’t Repeat Yourself)**:
   - Reuse code where possible. Extract reusable logic into libraries, helper methods, or utilities.
5. **Encapsulation**:
   - Minimize global access and reduce coupling between components.
6. **Error Handling**:
   - Handle exceptions gracefully. Use `try-catch` blocks only where necessary.
7. **Avoid Magic Numbers**:
   - Replace numerical constants or `string` literals used directly in code with named constants.
8. **Avoid unnecessary changes**:
   - Do not modify code or file unless strictly necessary and explain any extra changes you make.
9. **Always plan before coding**:
   - Before writing code, outline your approach and design to ensure a clear path forward.
   - Stop after planning and ask for feedback and approval before proceeding to implementation.
10. **Composition over Inheritance**:
    - Favor composition (using objects) over inheritance to achieve code reuse and flexibility.
11. **Use Interfaces**:
    - Define clear contracts for your classes using interfaces to promote loose coupling and testability.
12. **Consistent Formatting**:
    - Follow consistent code formatting and styling guidelines (e.g., spacing, indentation, line breaks) to enhance readability.
13. **Use Meaningful Names**:
    - Choose descriptive names for variables, methods, classes, and namespaces to clearly convey their purpose and intent.
14. **Write Unit Tests**:
    - Ensure that your code is testable and write unit tests to validate the functionality and catch regressions early.
---

## **2. Naming Conventions**

### **2.1 General Rules**
- Names should always convey the purpose of the variable, class, or method clearly.
- Use **camelCase** for local variables and method parameters.
- Use **_camelCase** for private fields.
- Use **PascalCase** for public properties, fields, methods, classes, constants, and namespaces.
- Avoid abbreviations unless they are commonly understood (e.g., `Id`, `Http`, `Xml`).

---

### **2.2 Naming Variables**

| **Scope**         | **Convention** | **Example**               |
|--------------------|----------------|---------------------------|
| Local variables    | camelCase      | `int count;`              |
| Fields (private)   | _camelCase     | `_counter;`               |
| Public fields      | PascalCase     | `public int TotalCount;`  |
| Constants          | UPPER_SNAKE_CASE | `const double PI = 3.14159;` |

- **Avoid single-character variable names** unless they're used in loops (e.g., `i`, `j`).
  
Example:
```csharp
var userId = 1234;
int totalCost = 250;
```

---

### **2.3 Naming Methods**
- Use **PascalCase** for method names.
- Method names should be verbs or actions.
- Reflect the purpose and behavior of the method.

Examples:
```csharp
public void SaveUser();
private int CalculateSum(int x, int y);
```

---

### **2.4 Naming Classes and Interfaces**
- Use **PascalCase** for class and interface names.
- Class names should be nouns and represent objects or concepts.
- Add meaningful suffixes like `Service`, `Manager`, or `Controller` to clarify responsibilities.

Examples:
```csharp
public class UserManager { }
public interface IAuditLogger { }
```

---

### **2.5 Naming Properties**
- **PascalCase** for property names.
- Properties should describe the data they encapsulate.

Examples:
```csharp
public string FirstName { get; set; }
public int Age { get; set; }
```

---

### **2.6 Naming Parameters**
- Use **camelCase** for method and constructor parameters.

Examples:
```csharp
public void SetName(string firstName, string lastName) { }
```

---

### **2.7 Naming Enums**
- Use **PascalCase** for enums.
- Enum members should be singular unless they represent flags.

Examples:
```csharp
public enum Status {
    Active,
    Inactive,
    Pending
}
```

---

## **3. Code Style Rules**

### **3.1 Use Expression-Bodied Syntax**
- Use expression-bodied members for concise logic.
```csharp
public int Square(int x) => x * x;
```

---

### **3.2 Null-Checking**
- Use C# 8+ null-coalescing assignments:
```csharp
userName ??= "Guest";
```

- Use `ArgumentNullException.ThrowIfNull` for parameter null-checks:
```csharp
public void Process(string data) {
    ArgumentNullException.ThrowIfNull(data);
}
```

---

### **3.3 Async & Task Guidelines**
- Always use `async/await` conventions for modern .NET development.
- Follow proper naming convention (add `Async` suffix to method names):
```csharp
public async Task<User> GetUserByIdAsync(int id) { }
```

---

### **3.4 Dependency Injection**
- Use Constructor Injection for dependencies.
```csharp
public class UserController {
    private readonly IUserService _userService;

    public UserController(IUserService userService) {
        _userService = userService;
    }
}
```

---

## **4. Commenting Guidelines**

### **4.1 XML Code Documentation**
- Use XML comments to describe the purpose of public methods, classes, and properties.
```csharp
/// <summary>
/// Gets a user by ID from the database.
/// </summary>
/// <param name="id">The ID of the user.</param>
/// <returns>The User object with the specified ID.</returns>
public User GetUserById(int id)
{
    throw new NotImplementedException();
}
```

### **4.2 Inline Comments**
- Use sparingly and only when the intent behind a line of code isn't clear.
```csharp
// Check if the user is older than 18
if (user.Age > 18) {
    Console.WriteLine("Adult user");
}
```

---

## **5. Unit Testing Practices**
- Write tests for all logical methods and business logic.
- Use **naming convention**:
  - `[MethodName]_[Scenario]_[ExpectedResult]`.
  - Example:
    ```csharp
    public void GetUserById_UserExists_ReturnsUser() { }
    ```

- Group tests logically:
  ```plaintext
  MyApp.Tests/
    Services/
    Repositories/
  ```

- Use mocking libraries (e.g., Moq) for dependencies.

---

## **6. Performance Optimization**
- Minimize memory allocations.
- Use asynchronous methods (`async-await`) for I/O-bound calls.
- Leverage **caching** for expensive or frequently requested data:
  - Example: Use **MemoryCache** or distributed caching such as **Redis**.

---

## **7. Security Best Practices**
- Validate **all user inputs** to avoid SQL injection and XSS.
- Do not expose sensitive data (e.g., connection strings) in the code. Use `appsettings.json` or secure environment variables for secrets.
- Use **HTTPS** for all network communication.

## **8. Code Reviews**
- Always conduct code reviews to maintain code quality and share knowledge.
- Only comment when you have HIGH CONFIDENCE (>80%) that an issue exists
- Be concise: one sentence per comment when possible
- Focus on the code, not the person: avoid personal attacks or assumptions about the developer's intentions
- Focus on actionable feedback, not observations: suggest specific changes or improvements rather than just pointing out issues
- Prioritize issues: focus on critical bugs or design flaws first
- Unsafe code blocks without justification
- Missing input validation or error handling
- Violations of SOLID principles or design patterns
- Inconsistent naming conventions or formatting
- Unnecessary complexity or over-engineering
- Lack of unit tests for critical code paths
- Race conditions or thread safety issues in concurrent code
- Performance bottlenecks or inefficient algorithms
- Security vulnerabilities such as SQL injection, XSS, or sensitive data exposure
- Incorrect use of async/await or blocking calls in asynchronous code
- Inappropriate use of global state or static variables that can lead to tight coupling and testing difficulties
- Unclear or misleading comments that do not accurately describe the code's behavior or intent
- Incorrect error propagation
- Unnecessary comments that restate obvious code behavior
- Missing logging or insufficient logging for critical operations
- Stay silent when you are unsure about an issue or when the code is correct but could be improved in a non-critical way. Focus on providing feedback that adds value and helps maintain high code quality without overwhelming the developer with minor issues or subjective opinions.
