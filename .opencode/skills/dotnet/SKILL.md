---
name: dotnet
description: .NET-specific additions for scaffold, test, document, deps-update, migrate, profile, and deploy skills — use alongside base skills when working in a .NET project
license: MIT
compatibility: opencode
metadata:
  tags: dotnet, csharp, mvc, aspnet
---

## What I do

Extend the base skills (`scaffold`, `test`, `document`, `deps-update`, `migrate`, `profile`, `deploy`) with .NET-specific knowledge:
`dotnet` CLI, ASP.NET MVC/Minimal API patterns, EF Core migrations, xUnit, XML docs, NuGet, and deployment to IIS/Azure.

---

## Scaffold — .NET additions

### Use the dotnet CLI

```bash
# New projects
dotnet new mvc -n MyApp             # ASP.NET MVC
dotnet new webapi -n MyApi          # Web API (minimal or controller-based)
dotnet new classlib -n MyApp.Core   # Class library
dotnet new xunit -n MyApp.Tests     # Test project

# Add to solution
dotnet sln add MyApp/MyApp.csproj
dotnet sln add MyApp.Tests/MyApp.Tests.csproj

# Add project reference
dotnet add MyApp.Tests reference MyApp/MyApp.csproj
```

### MVC Controller pattern

```csharp
// Controllers/UsersController.cs
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly IUserService _userService;

    public UsersController(IUserService userService)
    {
        _userService = userService;
    }

    [HttpGet("{id}")]
    public async Task<ActionResult<UserDto>> GetById(Guid id)
    {
        var user = await _userService.GetByIdAsync(id);
        if (user is null) return NotFound();
        return Ok(user);
    }

    [HttpPost]
    public async Task<ActionResult<UserDto>> Create(CreateUserRequest request)
    {
        var user = await _userService.CreateAsync(request);
        return CreatedAtAction(nameof(GetById), new { id = user.Id }, user);
    }
}
```

### Wiring checklist for new artifacts

```csharp
// Program.cs — register services in DI container
builder.Services.AddScoped<IUserService, UserService>();
builder.Services.AddScoped<IUserRepository, UserRepository>();

// Add controller routing (MVC/API)
builder.Services.AddControllers();
app.MapControllers();

// Add Minimal API endpoint (alternative to controllers)
app.MapGet("/api/users/{id}", async (Guid id, IUserService svc) =>
    await svc.GetByIdAsync(id) is User u ? Results.Ok(u) : Results.NotFound());
```

---

## Test — .NET additions

### Project setup

```bash
dotnet add package xunit
dotnet add package xunit.runner.visualstudio
dotnet add package Moq
dotnet add package FluentAssertions
dotnet add package Microsoft.AspNetCore.Mvc.Testing  # for integration tests
```

### Unit test pattern (xUnit + Moq + FluentAssertions)

```csharp
public class UserServiceTests
{
    private readonly Mock<IUserRepository> _repoMock = new();
    private readonly UserService _sut;

    public UserServiceTests()
    {
        _sut = new UserService(_repoMock.Object);
    }

    [Fact]
    public async Task GetByIdAsync_ReturnsNull_WhenUserDoesNotExist()
    {
        _repoMock.Setup(r => r.FindByIdAsync(It.IsAny<Guid>()))
                 .ReturnsAsync((User?)null);

        var result = await _sut.GetByIdAsync(Guid.NewGuid());

        result.Should().BeNull();
    }

    [Theory]
    [InlineData("")]
    [InlineData(null)]
    [InlineData("  ")]
    public async Task CreateAsync_ThrowsArgumentException_WhenNameIsEmpty(string? name)
    {
        var act = async () => await _sut.CreateAsync(new CreateUserRequest { Name = name! });

        await act.Should().ThrowAsync<ArgumentException>()
                          .WithMessage("*name*");
    }
}
```

### Integration test pattern (WebApplicationFactory)

```csharp
public class UsersApiTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;

    public UsersApiTests(WebApplicationFactory<Program> factory)
    {
        _client = factory.WithWebHostBuilder(builder =>
        {
            builder.ConfigureServices(services =>
            {
                // Replace real DB with in-memory for tests
                services.RemoveAll<DbContextOptions<AppDbContext>>();
                services.AddDbContext<AppDbContext>(o => o.UseInMemoryDatabase("TestDb"));
            });
        }).CreateClient();
    }

    [Fact]
    public async Task GetUser_Returns404_WhenNotFound()
    {
        var response = await _client.GetAsync($"/api/users/{Guid.NewGuid()}");
        response.StatusCode.Should().Be(HttpStatusCode.NotFound);
    }
}
```

### Run tests

```bash
dotnet test                          # all tests
dotnet test --filter "FullyQualifiedName~UserService"  # filter by name
dotnet test --logger "console;verbosity=detailed"
dotnet test --collect:"XPlat Code Coverage"  # coverage report
```

---

## Document — .NET additions

### XML doc comments (standard for .NET — used by IntelliSense and Swagger)

```csharp
/// <summary>
/// Retrieves a user by their unique identifier.
/// </summary>
/// <param name="id">The user's GUID.</param>
/// <returns>
/// The user if found; <see langword="null"/> if no user with that ID exists.
/// </returns>
/// <exception cref="RepositoryException">
/// Thrown when the database connection fails.
/// </exception>
/// <example>
/// <code>
/// var user = await userService.GetByIdAsync(Guid.Parse("abc-123"));
/// if (user is null) return NotFound();
/// </code>
/// </example>
public async Task<User?> GetByIdAsync(Guid id)

/// <summary>The user's display name. Required; max 100 characters.</summary>
[Required, MaxLength(100)]
public string Name { get; set; } = string.Empty;
```

### Swagger/OpenAPI annotations (Swashbuckle)

```csharp
/// <summary>Get a user by ID.</summary>
/// <response code="200">Returns the user.</response>
/// <response code="404">User not found.</response>
[HttpGet("{id}")]
[ProducesResponseType(typeof(UserDto), 200)]
[ProducesResponseType(404)]
public async Task<ActionResult<UserDto>> GetById(Guid id) { ... }
```

Enable XML docs in `.csproj`:
```xml
<PropertyGroup>
  <GenerateDocumentationFile>true</GenerateDocumentationFile>
  <NoWarn>$(NoWarn);1591</NoWarn>  <!-- suppress missing XML doc warnings for non-public members -->
</PropertyGroup>
```

---

## Deps update — .NET / NuGet additions

```bash
# List outdated packages
dotnet list package --outdated
dotnet list package --outdated --include-transitive  # include transitive deps

# Install a specific version
dotnet add package Newtonsoft.Json --version 13.0.3

# Update to latest (edit .csproj or use):
dotnet add package Newtonsoft.Json  # installs latest compatible version

# Security audit
dotnet list package --vulnerable
dotnet list package --vulnerable --include-transitive
```

`.csproj` version pinning:
```xml
<PackageReference Include="Microsoft.EntityFrameworkCore" Version="8.0.*" />  <!-- patch auto-update -->
<PackageReference Include="Microsoft.EntityFrameworkCore" Version="8.*" />    <!-- minor auto-update -->
```

---

## Migrate — EF Core additions

```bash
# Install EF Core tools (once per machine)
dotnet tool install --global dotnet-ef

# Create a migration
dotnet ef migrations add AddUserAvatarUrl --project MyApp.Infrastructure --startup-project MyApp

# Apply to database
dotnet ef database update --project MyApp.Infrastructure --startup-project MyApp

# Roll back to a specific migration
dotnet ef database update PreviousMigrationName

# Generate SQL script (for production — don't run dotnet ef update directly in prod)
dotnet ef migrations script --idempotent -o migrations.sql
```

### Migration file structure

```csharp
// Migrations/20260416120000_AddUserAvatarUrl.cs
public partial class AddUserAvatarUrl : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.AddColumn<string>(
            name: "AvatarUrl",
            table: "Users",
            type: "nvarchar(500)",
            nullable: true);
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropColumn(name: "AvatarUrl", table: "Users");
    }
}
```

### Production migration rule

**Never** run `dotnet ef database update` directly against production.
Generate an idempotent SQL script, review it, then run via your deployment pipeline:
```bash
dotnet ef migrations script --idempotent --output deploy/migrations.sql
# Review migrations.sql, then apply via SQL client or pipeline step
```

---

## Profile — .NET additions

```bash
# Install diagnostic tools (once per machine)
dotnet tool install --global dotnet-trace
dotnet tool install --global dotnet-counters
dotnet tool install --global dotnet-dump

# Live counters (CPU, GC, request rate, exception rate)
dotnet counters monitor --process-id <pid> --counters System.Runtime,Microsoft.AspNetCore.Hosting

# CPU trace — collect
dotnet trace collect --process-id <pid> --output trace.nettrace
# Open trace.nettrace in PerfView or Speedscope (speedscope.app)

# Memory — collect heap dump
dotnet dump collect --process-id <pid>
dotnet dump analyze core.dmp   # then: dumpheap -stat, gcroot <address>
```

### Reading dotnet-counters output

```
[System.Runtime]
    GC Heap Size (MB)                        245       ← high? look for memory leaks
    Gen 0 GC / sec                            12
    Gen 2 GC / sec                             3       ← high Gen2 = long-lived allocations
    Exception Count / sec                     48       ← exceptions should be near 0 in steady state
    ThreadPool Queue Length                    0

[Microsoft.AspNetCore.Hosting]
    Current Requests                           8
    Requests / sec                           320
    Failed Requests / sec                      2       ← investigate if > 0
```

### Application Insights (Azure)

For apps deployed to Azure, use Application Insights for production profiling:
- **Live Metrics**: real-time request rate, failure rate, latency
- **Performance**: slowest requests with sampled traces
- **Profiler**: attach to a live App Service instance for CPU flame graphs

---

## Deploy — .NET additions

### Build for production

```bash
dotnet publish MyApp/MyApp.csproj \
  --configuration Release \
  --runtime linux-x64 \          # or win-x64, osx-x64
  --self-contained false \        # true = bundle runtime, larger binary
  --output ./publish
```

### Deployment targets

**IIS (Windows)**
```bash
# Install the ASP.NET Core Hosting Bundle on the server
# In IIS Manager: create site → set physical path to publish folder
# Application pool: No Managed Code, enable 32-bit if needed
# web.config is auto-generated by dotnet publish
```

**Azure App Service**
```bash
# Via Azure CLI
az webapp deploy --resource-group myRg --name myApp --src-path publish/

# Via GitHub Actions (typical pipeline step)
- uses: azure/webapps-deploy@v2
  with:
    app-name: myApp
    package: ./publish
```

**Docker**
```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY ["MyApp/MyApp.csproj", "MyApp/"]
RUN dotnet restore
COPY . .
RUN dotnet publish MyApp/MyApp.csproj -c Release -o /app/publish

FROM base AS final
COPY --from=build /app/publish .
ENTRYPOINT ["dotnet", "MyApp.dll"]
```

### `dotnet watch` for local development

```bash
dotnet watch run --project MyApp   # hot reload on file save
dotnet watch test                  # re-run tests on file save
```

### Health checks

```csharp
// Program.cs
builder.Services.AddHealthChecks()
    .AddDbContextCheck<AppDbContext>();

app.MapHealthChecks("/healthz");
```

Post-deploy smoke test: `curl https://myapp.example.com/healthz` should return `Healthy`.

---

## Modern C# patterns

### Records — use for immutable data/DTOs

```csharp
// Positional record (concise)
public record UserDto(Guid Id, string Name, string Email);

// Record with validation
public record CreateUserRequest(string Name, string Email)
{
    public string Name { get; init; } = string.IsNullOrWhiteSpace(Name)
        ? throw new ArgumentException("Name required", nameof(Name))
        : Name;
}

// Records support non-destructive mutation
var updated = original with { Name = "New Name" };
```

### Primary constructors (C# 12 / .NET 8+)

```csharp
// Service with primary constructor — replaces field + constructor boilerplate
public class UserService(IUserRepository repo, ILogger<UserService> logger)
{
    public async Task<User?> GetByIdAsync(Guid id)
    {
        logger.LogInformation("Fetching user {Id}", id);
        return await repo.FindByIdAsync(id);
    }
}
```

### Pattern matching

```csharp
// Switch expression
string Describe(object obj) => obj switch
{
    User u when u.IsAdmin  => $"Admin: {u.Name}",
    User u                 => $"User: {u.Name}",
    null                   => "null",
    _                      => "unknown"
};

// Property pattern in if
if (result is { IsSuccess: true, Value: var value })
    return Ok(value);
```

---

## Cancellation tokens

Always propagate `CancellationToken` through async call chains:

```csharp
// Controller — ASP.NET binds the request's token automatically
[HttpGet("{id}")]
public async Task<ActionResult<UserDto>> GetById(Guid id, CancellationToken ct)
{
    var user = await _userService.GetByIdAsync(id, ct);
    return user is null ? NotFound() : Ok(user);
}

// Service — pass token to repository and HttpClient
public async Task<User?> GetByIdAsync(Guid id, CancellationToken ct = default)
    => await _repo.FindByIdAsync(id, ct);

// Repository — pass to EF Core
public async Task<User?> FindByIdAsync(Guid id, CancellationToken ct)
    => await _db.Users.FirstOrDefaultAsync(u => u.Id == id, ct);
```

---

## IHttpClientFactory — typed clients

Never use `new HttpClient()` directly. Use the factory to manage lifetimes and avoid socket exhaustion:

```csharp
// Define a typed client
public class PaymentClient(HttpClient http)
{
    public async Task<PaymentResult> ChargeAsync(ChargeRequest req, CancellationToken ct)
    {
        var response = await http.PostAsJsonAsync("/charge", req, ct);
        response.EnsureSuccessStatusCode();
        return await response.Content.ReadFromJsonAsync<PaymentResult>(ct)
            ?? throw new InvalidOperationException("Empty response");
    }
}

// Register in Program.cs
builder.Services.AddHttpClient<PaymentClient>(client =>
{
    client.BaseAddress = new Uri("https://payments.example.com");
    client.Timeout = TimeSpan.FromSeconds(10);
});
```

---

## Options pattern

Bind configuration sections to strongly-typed classes:

```csharp
// appsettings.json
// { "Email": { "SmtpHost": "smtp.example.com", "Port": 587 } }

public record EmailOptions
{
    public const string Section = "Email";
    public required string SmtpHost { get; init; }
    public int Port { get; init; } = 587;
}

// Program.cs
builder.Services.AddOptions<EmailOptions>()
    .BindConfiguration(EmailOptions.Section)
    .ValidateDataAnnotations()
    .ValidateOnStart();

// Inject with IOptions<T>
public class EmailService(IOptions<EmailOptions> opts)
{
    private readonly EmailOptions _config = opts.Value;
}
```

---

## Middleware and global exception handling

### Custom middleware

```csharp
// Middleware/RequestLoggingMiddleware.cs
public class RequestLoggingMiddleware(RequestDelegate next, ILogger<RequestLoggingMiddleware> logger)
{
    public async Task InvokeAsync(HttpContext context)
    {
        logger.LogInformation("{Method} {Path}", context.Request.Method, context.Request.Path);
        await next(context);
        logger.LogInformation("Response {StatusCode}", context.Response.StatusCode);
    }
}

// Program.cs
app.UseMiddleware<RequestLoggingMiddleware>();
```

### Global exception handler (.NET 8+)

```csharp
// ExceptionHandling/GlobalExceptionHandler.cs
public class GlobalExceptionHandler(ILogger<GlobalExceptionHandler> logger) : IExceptionHandler
{
    public async ValueTask<bool> TryHandleAsync(HttpContext ctx, Exception ex, CancellationToken ct)
    {
        logger.LogError(ex, "Unhandled exception");

        var (status, title) = ex switch
        {
            ArgumentException   => (400, "Bad Request"),
            UnauthorizedAccessException => (401, "Unauthorized"),
            KeyNotFoundException => (404, "Not Found"),
            _                   => (500, "Internal Server Error"),
        };

        await ctx.Response.WriteAsJsonAsync(new { title, status }, ct);
        ctx.Response.StatusCode = status;
        return true;
    }
}

// Program.cs
builder.Services.AddExceptionHandler<GlobalExceptionHandler>();
builder.Services.AddProblemDetails();
app.UseExceptionHandler();
```
