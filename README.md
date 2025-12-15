# OliveGameStudio.MissingAttributes

A C# library providing compiler polyfills for modern language features in .NET Framework and .NET Standard. This library enables you to use the latest C# language features like records, init properties, and required members in projects targeting older frameworks.

## Installation

```bash
# Package manager
Install-Package OliveGameStudio.MissingAttributes

# .NET CLI
dotnet add package OliveGameStudio.MissingAttributes
```

## What are Compiler Polyfills?

Compiler polyfills are types that enable newer C# language features to work with older .NET runtime versions like .NET Framework. The C# compiler looks for specific types and attributes to enable language features. By providing these types, you can use the latest C# syntax even when targeting .NET Framework or .NET Standard.

## Included Polyfills

### `IsExternalInit`

Enables the `init` accessor for properties, allowing you to create immutable objects and records.

**Enables:**
- `init` property setters
- Records with `init` properties
- Object initializers with `init` properties

```csharp
// Without polyfill: Only works in .NET 5+
// With polyfill: Works in .NET Standard 2.0+

public class Person
{
    public string Name { get; init; }
    public int Age { get; init; }
}

// Usage
var person = new Person 
{ 
    Name = "John Doe", 
    Age = 30 
};

// person.Name = "Jane"; // Compile error - init properties are read-only after initialization
```

### `RequiredMemberAttribute`

Enables the `required` modifier for properties and fields, ensuring they must be initialized during object creation.

**Enables:**
- Required properties and fields
- Compile-time enforcement of initialization
- Better API design with mandatory parameters

```csharp
// Without polyfill: Only works in .NET 7+
// With polyfill: Works in .NET Standard 2.0+

public class User
{
    public required string Name { get; init; }
    public required string Email { get; init; }
    public string? PhoneNumber { get; init; } // Optional
}

// Usage
var user = new User
{
    Name = "John Doe",    // Required - compiler error if missing
    Email = "john@example.com" // Required - compiler error if missing
    // PhoneNumber is optional
};

// var invalid = new User { Name = "John" }; // Compile error - Email is required
```

### `SetsRequiredMembersAttribute`

Indicates that a constructor sets all required members, allowing object creation without object initializers.

**Enables:**
- Constructors that satisfy required member constraints
- Clean APIs that don't force object initializer syntax
- Better encapsulation with constructor-based initialization

```csharp
public class User
{
    public required string Name { get; init; }
    public required string Email { get; init; }
    public string? PhoneNumber { get; init; }

    // This constructor sets all required members
    [SetsRequiredMembers]
    public User(string name, string email)
    {
        Name = name;
        Email = email;
    }
    
    // Parameterless constructor still requires object initializer
    public User() { }
}

// Usage - both are valid:
var user1 = new User("John Doe", "john@example.com"); // Constructor sets required members
var user2 = new User { Name = "Jane", Email = "jane@example.com" }; // Object initializer
```

### `CompilerFeatureRequiredAttribute`

Indicates that compiler support for specific features is required. This attribute helps with advanced scenarios and compiler feature detection.

**Properties:**
- `FeatureName`: The name of the required compiler feature
- `IsOptional`: Whether the compiler can allow access if it doesn't understand the feature

**Built-in Feature Names:**
- `RefStructs`: For ref struct types
- `RequiredMembers`: For required members feature

```csharp
// Example usage (advanced scenarios)
[CompilerFeatureRequired(CompilerFeatureRequiredAttribute.RefStructs)]
public ref struct MyRefStruct
{
    public int Value;
}

[CompilerFeatureRequired(CompilerFeatureRequiredAttribute.RequiredMembers)]
public class MyClass
{
    public required string Name { get; init; }
}
```

## Usage Examples

### Records (Enabled by IsExternalInit)

```csharp
// Simple record
public record User(string Name, string Email);

// Record with additional properties
public record Product
{
    public string Name { get; init; }
    public decimal Price { get; init; }
    public string Category { get; init; }
    
    public Product(string name, decimal price)
    {
        Name = name;
        Price = price;
    }
}

// Usage
var user = new User("Alice", "alice@example.com");
var product = new Product("Laptop", 999.99m) { Category = "Electronics" };

// Records provide value equality
var user2 = new User("Alice", "alice@example.com");
Console.WriteLine(user == user2); // True

// Records support with expressions for immutable updates
var updatedUser = user with { Email = "alice.smith@example.com" };
```

### Init-Only Properties

```csharp
public class Configuration
{
    public string ConnectionString { get; init; }
    public int TimeoutSeconds { get; init; } = 30;
    public bool EnableLogging { get; init; } = true;
}

// Usage
var config = new Configuration
{
    ConnectionString = "Server=localhost;Database=MyDb",
    TimeoutSeconds = 60,
    EnableLogging = false
};

// config.ConnectionString = "new value"; // Compile error
```

### Immutable Collections

```csharp
public class GameState
{
    public IReadOnlyList<Player> Players { get; init; } = Array.Empty<Player>();
    public string CurrentLevel { get; init; }
    public int Score { get; init; }
    public DateTime StartTime { get; init; } = DateTime.UtcNow;
}

// Usage
var gameState = new GameState
{
    Players = new[] { new Player("Alice"), new Player("Bob") },
    CurrentLevel = "Level 1",
    Score = 0
};

// Create new state (immutable update pattern)
var newGameState = gameState with 
{ 
    Score = gameState.Score + 100,
    CurrentLevel = "Level 2"
};
```

### Required Members with Constructors

```csharp
public class DatabaseConnection
{
    public required string ConnectionString { get; init; }
    public required string Database { get; init; }
    public int TimeoutSeconds { get; init; } = 30;
    
    // Constructor that sets required members
    [SetsRequiredMembers]
    public DatabaseConnection(string connectionString, string database)
    {
        ConnectionString = connectionString;
        Database = database;
    }
    
    // Parameterless constructor for frameworks/serializers
    public DatabaseConnection() { }
}

// Both approaches work:
var conn1 = new DatabaseConnection("Server=localhost", "MyApp");
var conn2 = new DatabaseConnection 
{ 
    ConnectionString = "Server=localhost", 
    Database = "MyApp",
    TimeoutSeconds = 60 
};
```

### Required Members with Records

```csharp
public record CreateUserRequest
{
    public required string Name { get; init; }
    public required string Email { get; init; }
    public int? Age { get; init; } // Optional
    public string? PhoneNumber { get; init; } // Optional
    
    // Constructor for cleaner API
    [SetsRequiredMembers]
    public CreateUserRequest(string name, string email)
    {
        Name = name;
        Email = email;
    }
    
    public CreateUserRequest() { } // For model binding
}

// Usage options:
var request1 = new CreateUserRequest("Alice Smith", "alice@example.com");
var request2 = new CreateUserRequest
{
    Name = "Alice Smith",     // Required
    Email = "alice@example.com", // Required
    Age = 25                  // Optional
};
```

### API Models

```csharp
public record CreateUserRequest
{
    public string Name { get; init; }
    public string Email { get; init; }
    public int? Age { get; init; }
}

public record UserResponse(
    int Id,
    string Name,
    string Email,
    DateTime CreatedAt);

// Controller usage
[HttpPost]
public async Task<UserResponse> CreateUser(CreateUserRequest request)
{
    // request properties are immutable after binding
    var user = await _userService.CreateAsync(request.Name, request.Email, request.Age);
    
    return new UserResponse(user.Id, user.Name, user.Email, user.CreatedAt);
}
```

## Target Framework Support

This library enables the latest C# features in:

- **.NET Framework 4.6.1** and later  
- **.NET Standard 2.0** and later
- **.NET Core 2.0** and later

**Primary Focus**: Bringing modern C# language features to .NET Framework projects that can't upgrade to newer .NET versions.

Features that become available:
- **C# 9.0**: Records, init properties
- **C# 11.0**: Required members

## Project Configuration

Add this to your project file to use modern C# versions with older frameworks:

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net48</TargetFramework> <!-- .NET Framework 4.8 -->
    <LangVersion>latest</LangVersion> <!-- Enable latest C# features -->
  </PropertyGroup>
  
  <PackageReference Include="OliveStudio.MissingAttributes" Version="1.0.0" />
</Project>
```

## Migration Example

**Before (Traditional Classes):**
```csharp
public class User
{
    public User(string name, string email)
    {
        Name = name ?? throw new ArgumentNullException(nameof(name));
        Email = email ?? throw new ArgumentNullException(nameof(email));
    }
    
    public string Name { get; }
    public string Email { get; }
    
    public override bool Equals(object obj) { /* boilerplate */ }
    public override int GetHashCode() { /* boilerplate */ }
    public override string ToString() { /* boilerplate */ }
}
```

**After (With Polyfills):**
```csharp
public record User(string Name, string Email);
```

## Benefits

- **Modern Syntax**: Use latest C# features in .NET Framework projects
- **No Runtime Dependency**: Works with existing .NET Framework applications
- **Backward Compatibility**: Deploy to .NET Framework while using modern C# code
- **Reduced Boilerplate**: Records eliminate equals, hash code, and toString implementations
- **Immutability**: Init properties help create immutable objects
- **Team Productivity**: Developers can use familiar modern C# syntax
- **Migration Path**: Easier to upgrade projects incrementally

## Best Practices

### 1. Use Records for Data Transfer Objects
```csharp
// API models
public record LoginRequest(string Username, string Password);
public record LoginResponse(string Token, DateTime ExpiresAt);

// Domain events
public record UserCreatedEvent(int UserId, string Email, DateTime CreatedAt);
```

### 2. Use SetsRequiredMembers for Clean APIs
```csharp
public class ApiConfiguration
{
    public required string BaseUrl { get; init; }
    public required string ApiKey { get; init; }
    public int TimeoutMs { get; init; } = 5000;
    
    [SetsRequiredMembers]
    public ApiConfiguration(string baseUrl, string apiKey)
    {
        BaseUrl = baseUrl;
        ApiKey = apiKey;
    }
    
    public ApiConfiguration() { } // For configuration binding
}
```
```csharp
public class DatabaseOptions
{
    public string ConnectionString { get; init; }
    public int CommandTimeout { get; init; } = 30;
    public bool EnableRetry { get; init; } = true;
}
```

### 3. Prefer Init Properties for Configuration
```csharp
public class DatabaseOptions
{
    public string ConnectionString { get; init; }
    public int CommandTimeout { get; init; } = 30;
    public bool EnableRetry { get; init; } = true;
}
```

### 4. Use With Expressions for Updates
```csharp
public record UserSettings(string Theme, bool Notifications, string Language)
{
    public UserSettings WithTheme(string newTheme) => this with { Theme = newTheme };
    public UserSettings WithNotifications(bool enabled) => this with { Notifications = enabled };
}
```

## Common Scenarios

### Value Objects in Domain-Driven Design
```csharp
public record Money(decimal Amount, string Currency)
{
    public Money Add(Money other)
    {
        if (Currency != other.Currency)
            throw new InvalidOperationException("Cannot add different currencies");
            
        return this with { Amount = Amount + other.Amount };
    }
}
```

### Immutable Game State
```csharp
public record GameState(
    int Level,
    int Score,
    TimeSpan TimeRemaining,
    IReadOnlyList<string> Inventory)
{
    public GameState AddScore(int points) => this with { Score = Score + points };
    public GameState NextLevel() => this with { Level = Level + 1, TimeRemaining = TimeSpan.FromMinutes(5) };
}
```
