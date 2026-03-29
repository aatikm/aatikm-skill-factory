---
name: dotnet-unit-test
description: Use when writing or reviewing unit tests in .NET 8 C# projects. Applies to test structure, naming, mocking, assertions, and framework setup using xUnit, Moq, and Microsoft.EntityFrameworkCore.InMemory.
---

# .NET 8 Unit Testing Style

## Stack

| Role | Library | Version |
|------|---------|---------|
| Test framework | xUnit | 2.5.3 |
| Mock framework | Moq | 4.20.72 |
| In-memory database | Microsoft.EntityFrameworkCore.InMemory | 8.0.8 |
| Test SDK | Microsoft.NET.Test.Sdk | 17.8.0 |
| Coverage | coverlet.collector | 6.0.0 |

### Package Reference (csproj)

```xml
<PackageReference Include="Microsoft.NET.Test.Sdk" Version="17.8.0" />
<PackageReference Include="xunit" Version="2.5.3" />
<PackageReference Include="xunit.runner.visualstudio" Version="2.5.3" />
<PackageReference Include="Moq" Version="4.20.72" />
<PackageReference Include="Microsoft.EntityFrameworkCore.InMemory" Version="8.0.8" />
<PackageReference Include="coverlet.collector" Version="6.0.0" />
```

---

## Rules at a Glance

| Rule | Description |
|------|-------------|
| One assertion focus per test | Each test validates one logical behavior |
| Naming: `Method_Scenario_ExpectedBehavior` | Test name is the specification |
| AAA structure | Arrange / Act / Assert separated by blank lines |
| Mock only external dependencies | Interfaces, HTTP clients — not domain objects or value objects |
| Use in-memory EF for DbContext | Prefer `UseInMemoryDatabase` over mocking DbContext directly when EF operations are tested end-to-end |
| Use `Moq` for interface mocking | `new Mock<IInterface>()`, access via `.Object` |
| Use `[Fact]` for single-case tests | Use `[Theory]` with `[InlineData]` for parameterized tests |
| Test public API only | Never test private or internal implementation details |
| Use `Create...` helper methods | Extract `new` expressions for inputs and SUT construction |
| Use `Given...` helper methods | Extract mock `.Setup(...)` calls that would clutter the Arrange block |
| Constructor for setup | xUnit creates a new instance per test; initialize mocks and SUT in the constructor |

---

## Test Structure

### Naming Convention

```
MethodName_Scenario_ExpectedBehavior
```

```csharp
// examples
Handle_ShouldInsertOnlyNonExistingResources
Handle_WhenAllResourcesExist_ShouldNotInsert
Handle_ReturnsExpectedTradePartnerResponses
GetTaggedMilestoneProjects_ReturnsEmptyList_WhenNoProjects
Handle_ReturnsMappedDisplayViews
```

### AAA Layout

```csharp
[Fact]
public async Task Handle_ShouldInsertAndDispatchEvent_WhenSubscriptionDoesNotExist()
{
    // Arrange
    var dbContext = CreateInMemoryDbContext();
    var eventDispatcherMock = new Mock<IDomainEventDispatcher>();
    var handler = new CreateEmailSubscriptionCommandHandler(dbContext, eventDispatcherMock.Object);
    var request = CreateSubscriptionCommand("test@example.com", 123);

    // Act
    var result = await handler.Handle(request, CancellationToken.None);

    // Assert
    Assert.Equal((1, 0), result);
}
```

---

## Setup

### Mocking with Moq

Declare mocks and the SUT as private fields and initialize them in the constructor. Pass `.Object` to the SUT.

```csharp
public class GetBuDisplayViewsQueryHandlerTests
{
    private readonly Mock<IResDbContext> _resDbContextMock;
    private readonly GetBuDisplayViewsQueryHandler _handler;

    public GetBuDisplayViewsQueryHandlerTests()
    {
        _resDbContextMock = new Mock<IResDbContext>();
        _handler = new GetBuDisplayViewsQueryHandler(_resDbContextMock.Object);
    }
}
```

> xUnit creates a new class instance per `[Fact]`, so field-level initialization in the constructor is test-isolated and safe.

### In-Memory DbContext

Use `UseInMemoryDatabase` with a unique name per test (via `Guid.NewGuid()`) to prevent state leakage between tests.

```csharp
private IResDbContext CreateInMemoryDbContext()
{
    var options = new DbContextOptionsBuilder<ResDbContext>()
        .UseInMemoryDatabase(databaseName: "TestDb_" + Guid.NewGuid())
        .Options;
    return new ResDbContext(options);
}
```

Use the shared project utilities for common DbContext types:

```csharp
// For ScheduleViewerDbContext
var dbContext = DbContextTestUtil.CreateInMemoryDbContext();

// For ConfigDbContext
var configDb = ConfigDbContextTestUtil.CreateInMemoryConfigDbContext();
```

### Mocking DbSet on a Mocked Interface

When the handler under test uses `IQueryable` / `async` EF operations on a mocked context interface (not a real DbContext), use the `SetupDbSet` extension and the `TestAsyncQueryProvider` utility:

```csharp
var data = new List<AssignedResource> { ... }.AsQueryable();
contextMock.SetupDbSet(c => c.AssignedResources, data);
```

The `SetupDbSet` helper (in `CommandTests\DbSetMockExtensions.cs`) wires up `IQueryable<T>`, `IAsyncEnumerable<T>`, and `Provider` so EF LINQ-to-objects async queries work correctly in tests.

For query handlers that need a per-test `DbSet` mock, use the inline `CreateMockDbSet` pattern:

```csharp
private static Mock<DbSet<T>> CreateMockDbSet<T>(List<T> data) where T : class
{
    var queryable = data.AsQueryable();
    var mockSet = new Mock<DbSet<T>>();
    mockSet.As<IAsyncEnumerable<T>>()
        .Setup(m => m.GetAsyncEnumerator(It.IsAny<CancellationToken>()))
        .Returns(new TestAsyncEnumerator<T>(queryable.GetEnumerator()));
    mockSet.As<IQueryable<T>>()
        .Setup(m => m.Provider)
        .Returns(new TestAsyncQueryProvider<T>(queryable.Provider));
    mockSet.As<IQueryable<T>>().Setup(m => m.Expression).Returns(queryable.Expression);
    mockSet.As<IQueryable<T>>().Setup(m => m.ElementType).Returns(queryable.ElementType);
    mockSet.As<IQueryable<T>>().Setup(m => m.GetEnumerator()).Returns(queryable.GetEnumerator());
    return mockSet;
}
```

---

## Folder Structure

```
Dpr.IP.ScheduleViewer.Test\
├── CommandTests\       # Tests for IRequestHandler command handlers
│   └── DbSetMockExtensions.cs    # SetupDbSet extension for Mock<IScheduleViewerDbContext>
├── QueryTests\         # Tests for IRequestHandler query handlers
├── ServiceTests\       # Tests for orchestrators and application services
├── Helpers\            # Shared test helpers and extensions
├── Models\             # Test-specific model builders / fixtures
└── Utility\
    ├── DbContextTestUtil.cs          # Creates in-memory ScheduleViewerDbContext
    ├── ConfigDbContextTestUtil.cs    # Creates in-memory ConfigDbContext
    └── TestAsyncQueryProvider.cs     # TestAsyncQueryProvider<T>, TestAsyncEnumerable<T>, TestAsyncEnumerator<T>
```

---

## Helper Methods

### `Given...` — mock setup

Extract `.Setup(...)` calls into `Given` methods when they would clutter the Arrange block.

```csharp
private void GivenRfiResponse(List<TPSnowflakeRfiResponse> responses)
{
    _snowflakeMock
        .Setup(x => x.ExecuteGenericDataObjectQueryAsync<TPSnowflakeRfiResponse>(
            It.IsAny<string>(), It.IsAny<string>(), "", null, false))
        .ReturnsAsync(responses);
}

private void GivenUserNotFound(Guid userId)
{
    _userRepoMock
        .Setup(r => r.GetAsync(userId, It.IsAny<CancellationToken>()))
        .ReturnsAsync((User?)null);
}
```

### `Create...` — object construction

Extract `new` expressions for commands, queries, and DTOs into `Create` methods.

```csharp
private static CreateEmailSubscriptionCommand CreateSubscriptionCommand(
    string email, int projectId, int intervalDays = 7) =>
    new(new List<EmailSubscriptionRequest>
    {
        new()
        {
            UserEmail = email,
            ProjectId = projectId,
            ProjectName = "Test Project",
            SubscriptionType = SubscriptionType.HRA,
            IntervalDays = intervalDays
        }
    });
```

---

## Assertions

Use xUnit's built-in `Assert` class.

```csharp
// equality
Assert.Equal(expected, actual);
Assert.Equal((1, 0), result);

// null checks
Assert.NotNull(result);
Assert.Null(result);

// boolean
Assert.True(condition);
Assert.False(condition);

// collections
Assert.Single(collection);
Assert.Empty(collection);
Assert.Contains(collection, x => x.Id == expectedId);
Assert.All(collection, x => Assert.NotNull(x.Name));
Assert.Equal(3, result.Count);

// async exception
await Assert.ThrowsAsync<Exception>(() => _sut.Handle(command, CancellationToken.None));

// assert on exception message
var ex = await Assert.ThrowsAsync<Exception>(() => _sut.Handle(command, CancellationToken.None));
Assert.Contains("not found", ex.Message);
```

### Moq Verification

```csharp
// called exactly once
mock.Verify(x => x.SaveChangesAsync(It.IsAny<CancellationToken>()), Times.Once);

// never called
mock.Verify(x => x.SaveChangesAsync(It.IsAny<CancellationToken>()), Times.Never);

// called with a specific argument type
mock.Verify(x => x.DispatchEventAsync(It.IsAny<ProjectSubscription>()), Times.Once);
```

---

## What to Mock

| Mock it (Moq) | Use In-Memory EF | Don't mock it |
|---|---|---|
| DbContext interfaces (`IScheduleViewerDbContext`, `IResDbContext`, `IConfigDbContext`) when only select/filter behavior matters | DbContext-backed handlers where full EF persistence (add / update / query) is under test | Domain models (`Assignment`, `EmailSubscription`) |
| HTTP clients (`ISnowflakeClient`, `IHubApiClient`, `IPcuClient`, `IEstimatorClient`) | | Value objects |
| `IMediator` (in orchestrator / service tests) | | Pure computation utilities |
| `ILoginAccountProvider` | | In-memory collections |
| `IDomainEventDispatcher` | | |
| `IIntegratedProjectsDbConnector` | | |

---

## Full Examples

### Command Handler — In-Memory EF (full persistence test)

```csharp
public class CreateEmailSubscriptionCommandHandlerTests
{
    private IResDbContext CreateInMemoryDbContext()
    {
        var options = new DbContextOptionsBuilder<ResDbContext>()
            .UseInMemoryDatabase(databaseName: "TestDb_" + Guid.NewGuid())
            .Options;
        return new ResDbContext(options);
    }

    [Fact]
    public async Task Handle_ShouldInsertAndDispatchEvent_WhenSubscriptionDoesNotExist()
    {
        // Arrange
        var dbContext = CreateInMemoryDbContext();
        var eventDispatcherMock = new Mock<IDomainEventDispatcher>();
        var handler = new CreateEmailSubscriptionCommandHandler(dbContext, eventDispatcherMock.Object);
        var request = CreateSubscriptionCommand("test@example.com", 123);

        // Act
        var result = await handler.Handle(request, CancellationToken.None);

        // Assert
        Assert.Equal((1, 0), result);
        var inserted = await dbContext.EmailSubscriptions.FirstOrDefaultAsync();
        Assert.NotNull(inserted);
        Assert.Equal("test@example.com", inserted.UserEmail);
        eventDispatcherMock.Verify(x => x.DispatchEventAsync(It.IsAny<ProjectSubscription>()), Times.Once);
    }

    // --- Create ---

    private static CreateEmailSubscriptionCommand CreateSubscriptionCommand(
        string email, int projectId, int intervalDays = 7) =>
        new(new List<EmailSubscriptionRequest>
        {
            new()
            {
                UserEmail = email,
                ProjectId = projectId,
                ProjectName = "Test Project",
                SubscriptionType = SubscriptionType.HRA,
                IntervalDays = intervalDays
            }
        });
}
```

### Command Handler — Mocked DbContext + SetupDbSet (filtered write test)

```csharp
public class InsertAssignedResourcesCommandHandlerTests
{
    [Fact]
    public async Task Handle_ShouldInsertOnlyNonExistingResources()
    {
        // Arrange
        var contextMock = new Mock<IScheduleViewerDbContext>();
        var loginMock = new Mock<ILoginAccountProvider>();
        loginMock.Setup(l => l.UserEmail).Returns("test@domain.com");

        var existingResources = new List<AssignedResource>
        {
            new() { AssessmentId = 1, AssessmentProjectId = "proj-1", UserId = "user-1", UserType = "TypeA" }
        }.AsQueryable();
        contextMock.SetupDbSet(c => c.AssignedResources, existingResources);
        contextMock
            .Setup(c => c.SaveChangesAsync(It.IsAny<CancellationToken>()))
            .ReturnsAsync(1);

        var handler = new InsertAssignedResourcesCommandHandler(contextMock.Object, loginMock.Object);
        var request = CreateAssignCommand();

        // Act
        var result = await handler.Handle(request, CancellationToken.None);

        // Assert
        Assert.Equal(1, result);
        contextMock.Verify(c => c.SaveChangesAsync(It.IsAny<CancellationToken>()), Times.Once);
    }

    // --- Create ---

    private static AssignPcaResourcesCommand CreateAssignCommand() =>
        new(new List<AssignPcaResourcesRequest>
        {
            new() { AssessmentId = 1, AssessmentProjectID = "proj-1", UserID = "user-1", UserType = "TypeA" },
            new() { AssessmentId = 2, AssessmentProjectID = "proj-2", UserID = "user-2", UserType = "TypeB" }
        });
}
```

### Query Handler — Mocked Interface + Mocked DbSet

```csharp
public class GetBuDisplayViewsQueryHandlerTests
{
    private readonly Mock<IResDbContext> _resDbContextMock;
    private readonly GetBuDisplayViewsQueryHandler _handler;

    public GetBuDisplayViewsQueryHandlerTests()
    {
        _resDbContextMock = new Mock<IResDbContext>();
        _handler = new GetBuDisplayViewsQueryHandler(_resDbContextMock.Object);
    }

    [Fact]
    public async Task Handle_ReturnsMappedDisplayViews()
    {
        // Arrange
        var mockSet = CreateMockDbSet(new List<LkBuDisplayView>
        {
            new() { Id = 1, ViewName = "View 1", ResourceTypeId = 10 }
        });
        _resDbContextMock.Setup(x => x.LkBuDisplayViews).Returns(mockSet.Object);

        // Act
        var result = await _handler.Handle(new GetBuDisplayViews(), CancellationToken.None);

        // Assert
        Assert.Single(result);
        Assert.Equal("View 1", result[0].Name);
    }

    [Fact]
    public async Task Handle_ReturnsEmptyList_WhenNoViewsExist()
    {
        // Arrange
        var mockSet = CreateMockDbSet(new List<LkBuDisplayView>());
        _resDbContextMock.Setup(x => x.LkBuDisplayViews).Returns(mockSet.Object);

        // Act
        var result = await _handler.Handle(new GetBuDisplayViews(), CancellationToken.None);

        // Assert
        Assert.Empty(result);
    }

    // --- Create ---

    private static Mock<DbSet<LkBuDisplayView>> CreateMockDbSet(List<LkBuDisplayView> data)
    {
        var queryable = data.AsQueryable();
        var mockSet = new Mock<DbSet<LkBuDisplayView>>();
        mockSet.As<IAsyncEnumerable<LkBuDisplayView>>()
            .Setup(m => m.GetAsyncEnumerator(It.IsAny<CancellationToken>()))
            .Returns(new TestAsyncEnumerator<LkBuDisplayView>(queryable.GetEnumerator()));
        mockSet.As<IQueryable<LkBuDisplayView>>()
            .Setup(m => m.Provider)
            .Returns(new TestAsyncQueryProvider<LkBuDisplayView>(queryable.Provider));
        mockSet.As<IQueryable<LkBuDisplayView>>().Setup(m => m.Expression).Returns(queryable.Expression);
        mockSet.As<IQueryable<LkBuDisplayView>>().Setup(m => m.ElementType).Returns(queryable.ElementType);
        mockSet.As<IQueryable<LkBuDisplayView>>().Setup(m => m.GetEnumerator()).Returns(queryable.GetEnumerator());
        return mockSet;
    }
}
```

### Service / Orchestrator — IMediator Mock

```csharp
public class MilestoneProjectOrchestratorTests
{
    [Fact]
    public async Task GetTaggedMilestoneProjects_ReturnsEmptyList_WhenNoProjects()
    {
        // Arrange
        var mediatorMock = new Mock<IMediator>();
        var orchestrator = new MilestoneProjectOrchestrator(mediatorMock.Object);

        // Act
        var result = await orchestrator.GetTaggedMilestoneProjects(
            new List<ProjectInfoByRegion>(), null, null, null);

        // Assert
        Assert.Empty(result);
    }

    [Fact]
    public async Task GetTaggedMilestoneProjects_ReturnsCombinedAndDeduplicatedProjects()
    {
        // Arrange
        var mediatorMock = new Mock<IMediator>();
        GivenP6Projects(mediatorMock, new List<BuProjectModel>
        {
            new() { OriginalProjectId = 1, ProjectId = 101 }
        });
        GivenHubMilestones(mediatorMock, new List<HubMilestone>
        {
            new() { OriginalProjectId = 2, JobNumber = "J002" }
        });
        var orchestrator = new MilestoneProjectOrchestrator(mediatorMock.Object);

        // Act
        var result = await orchestrator.GetTaggedMilestoneProjects(
            CreateProjectsByRegion(), null, new[] { "P6A" }, new[] { "HUB1" });

        // Assert
        Assert.Equal(2, result.Count);
        Assert.Contains(result, p => p.OriginalProjectId == 1);
        Assert.Contains(result, p => p.OriginalProjectId == 2);
    }

    // --- Given ---

    private static void GivenP6Projects(Mock<IMediator> mock, List<BuProjectModel> projects) =>
        mock.Setup(m => m.Send(It.IsAny<GetProjectsByCategoriesQuery>(), default))
            .ReturnsAsync(projects);

    private static void GivenHubMilestones(Mock<IMediator> mock, List<HubMilestone> milestones) =>
        mock.Setup(m => m.Send(It.IsAny<GetHubMilestonesByProjectQuery>(), default))
            .ReturnsAsync(milestones);

    // --- Create ---

    private static List<ProjectInfoByRegion> CreateProjectsByRegion() =>
    [
        new() { OriginalProjectId = 1, JobNumber = "J001", ProjectName = "Project One", SchedulerName = "SchedulerA" },
        new() { OriginalProjectId = 2, JobNumber = "J002", ProjectName = "Project Two", SchedulerName = "SchedulerB" }
    ];
}
```

---

## Utility Classes (Utility\)

These are shared across the test project and must not be duplicated per test file.

| Class | Location | Purpose |
|---|---|---|
| `DbContextTestUtil` | `Utility\DbContextTestUtil.cs` | Creates an isolated in-memory `ScheduleViewerDbContext` |
| `ConfigDbContextTestUtil` | `Utility\ConfigDbContextTestUtil.cs` | Creates an isolated in-memory `ConfigDbContext` |
| `TestAsyncQueryProvider<T>` | `Utility\TestAsyncQueryProvider.cs` | Implements `IAsyncQueryProvider` so EF async LINQ works on a mocked `DbSet` |
| `TestAsyncEnumerable<T>` | `Utility\TestAsyncQueryProvider.cs` | Wraps `IEnumerable<T>` for async EF enumeration |
| `TestAsyncEnumerator<T>` | `Utility\TestAsyncQueryProvider.cs` | Implements `IAsyncEnumerator<T>` backed by a synchronous enumerator |
| `DbSetMockExtensions.SetupDbSet` | `CommandTests\DbSetMockExtensions.cs` | Extension on `Mock<IScheduleViewerDbContext>` to wire a full async-queryable mocked `DbSet<T>` in one call |

---

## Violation Quick Reference

| Symptom | Fix |
|---|---|
| Multiple unrelated `Assert` calls in one test | Split into separate `[Fact]` methods |
| Mock setup inline cluttering Arrange | Extract to `Given...` method |
| `new SomeDto(...)` repeated across tests | Extract to `Create...` method |
| `UseInMemoryDatabase` with a fixed string name | Use `Guid.NewGuid().ToString()` to isolate tests |
| Mocking EF DbSet for simple CRUD a real DbContext handles | Use `UseInMemoryDatabase` instead |
| Async EF queries failing on mocked `DbSet` | Use `SetupDbSet` extension or inline `CreateMockDbSet` helper with `TestAsyncQueryProvider` |
| Manually duplicating async DbSet boilerplate per test class | Extract to a `CreateMockDbSet` private helper or use `DbSetMockExtensions.SetupDbSet` |
| Test name is `Test1` or `ShouldWork` | Rename to `Method_Scenario_ExpectedBehavior` |
| Mocking a domain model or value object | Use a real instance instead |
| Missing blank lines between AAA blocks | Add blank lines to separate Arrange / Act / Assert |
| Verifying a call that was never set up | Add `.Setup(...)` before `.Verify(...)` |
