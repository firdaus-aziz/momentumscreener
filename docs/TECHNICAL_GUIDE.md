# Technical Guide

Architecture and implementation details for the Momentum Screener.

## Architecture Overview

### Dual-Language Design

| Layer | Technology | Responsibility |
|-------|------------|----------------|
| **UI** | Blazor Server + MudBlazor | Dashboard, real-time updates |
| **Application** | ASP.NET Core | Services, background jobs |
| **Domain** | C# (.NET 8) | Entities, business logic |
| **Infrastructure** | EF Core + SQLite | Persistence, external APIs |
| **Data Acquisition** | Python | TradingView, yfinance |

### Project Structure

```
momentum-screener/
├── src/
│   ├── MomentumScreener.Core/           # Pure domain, no dependencies
│   ├── MomentumScreener.Infrastructure/ # EF Core, services
│   └── MomentumScreener.Web/            # Blazor Server
├── tests/
│   ├── MomentumScreener.Core.Tests/     # Unit tests
│   └── MomentumScreener.Integration.Tests/
├── tools/py/                            # Python scripts
└── data/                                # SQLite database
```

### Dependency Flow

```
MomentumScreener.Web
         │
         ▼
MomentumScreener.Infrastructure
         │
         ▼
MomentumScreener.Core (no external deps)
```

---

## Core Domain

### Entities

#### Alert

```csharp
public class Alert
{
    public Guid Id { get; set; }
    public string Ticker { get; set; } = string.Empty;
    public AlertType AlertType { get; set; }
    public decimal Price { get; set; }
    public long Volume { get; set; }
    public decimal RelativeVolume { get; set; }
    public string Sector { get; set; } = string.Empty;
    public decimal ChangePercent { get; set; }
    public decimal ChangeFromOpen { get; set; }
    public DateTime Timestamp { get; set; }

    // Flat-to-spike analysis
    public bool IsFlat { get; set; }
    public decimal FlatDurationMinutes { get; set; }
    public decimal FlatVolatility { get; set; }

    // Navigation
    public ICollection<Position> Positions { get; set; } = new List<Position>();
}
```

#### Position

```csharp
public class Position
{
    public Guid Id { get; set; }
    public string Ticker { get; set; } = string.Empty;
    public decimal EntryPrice { get; set; }
    public decimal CurrentPrice { get; set; }
    public decimal Shares { get; set; }
    public decimal PositionSize { get; set; }
    public DateTime EntryTime { get; set; }
    public PositionStatus Status { get; set; }

    // Foreign keys
    public Guid AlertId { get; set; }
    public Alert Alert { get; set; } = null!;

    // Navigation
    public Trade? Trade { get; set; }
}
```

#### Trade

```csharp
public class Trade
{
    public Guid Id { get; set; }
    public string Ticker { get; set; } = string.Empty;
    public decimal EntryPrice { get; set; }
    public decimal ExitPrice { get; set; }
    public decimal Shares { get; set; }
    public DateTime EntryTime { get; set; }
    public DateTime ExitTime { get; set; }
    public decimal ProfitLoss { get; set; }
    public decimal ProfitPercent { get; set; }
    public TradeExitReason ExitReason { get; set; }
    public AlertType AlertType { get; set; }

    // Foreign key
    public Guid PositionId { get; set; }
    public Position Position { get; set; } = null!;
}
```

### Enums

```csharp
public enum AlertType
{
    FlatToSpike,
    PriceSpike,
    VolumeClimber,
    VolumeNewcomer,
    PremarketPrice,
    PremarketVolume
}

public enum PositionStatus
{
    Pending,
    Open,
    Closed
}

public enum TradeExitReason
{
    EmaExit,
    StopLoss,
    TakeProfit,
    Manual,
    EndOfDay
}

public enum AutomationPhase
{
    PaperTrading,
    ManualExecute,
    OneClick,
    AutoWithApproval,
    FullAuto
}

public enum MarketConditionCategory
{
    Excellent,  // 80-100
    Good,       // 65-79
    Fair,       // 45-64
    Poor        // 0-44
}
```

### Interfaces

```csharp
public interface IAlertRepository
{
    Task<Alert?> GetByIdAsync(Guid id);
    Task<IEnumerable<Alert>> GetRecentAsync(int count);
    Task<IEnumerable<Alert>> GetByTypeAsync(AlertType type);
    Task AddAsync(Alert alert);
    Task SaveChangesAsync();
}

public interface ITradingEngine
{
    Task ProcessAlertAsync(Alert alert);
    bool ShouldEnterPosition(Alert alert, MarketCondition condition);
    bool ShouldExitPosition(Position position, decimal currentPrice);
}

public interface IRiskManager
{
    decimal CalculatePositionSize(Alert alert, RiskProfile profile, decimal accountBalance);
    bool CanOpenPosition(RiskProfile profile, IEnumerable<Position> openPositions);
    decimal GetCurrentExposure(IEnumerable<Position> positions);
    bool IsDailyLossLimitReached(IEnumerable<Trade> todayTrades, RiskProfile profile, decimal accountBalance);
}

public interface IPythonBridge
{
    Task<IEnumerable<Alert>> FetchAlertsAsync();
    Task<MarketCondition> FetchMarketConditionAsync();
    Task<decimal> FetchCurrentPriceAsync(string ticker);
}
```

---

## Infrastructure

### Database Context

```csharp
public class AppDbContext : DbContext
{
    public DbSet<Alert> Alerts => Set<Alert>();
    public DbSet<Position> Positions => Set<Position>();
    public DbSet<Trade> Trades => Set<Trade>();
    public DbSet<RiskProfile> RiskProfiles => Set<RiskProfile>();

    protected override void OnConfiguring(DbContextOptionsBuilder options)
    {
        options.UseSqlite("Data Source=data/momentum.db");
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Configure relationships
        modelBuilder.Entity<Position>()
            .HasOne(p => p.Alert)
            .WithMany(a => a.Positions)
            .HasForeignKey(p => p.AlertId);

        modelBuilder.Entity<Trade>()
            .HasOne(t => t.Position)
            .WithOne(p => p.Trade)
            .HasForeignKey<Trade>(t => t.PositionId);
    }
}
```

### Python Bridge

```csharp
public class PythonBridge : IPythonBridge
{
    private readonly PythonSettings _settings;

    public PythonBridge(IOptions<PythonSettings> settings)
    {
        _settings = settings.Value;
    }

    public async Task<IEnumerable<Alert>> FetchAlertsAsync()
    {
        string json = await ExecuteScriptAsync("tools/py/screener/fetch_alerts.py", "--json");
        return JsonSerializer.Deserialize<List<Alert>>(json) ?? new List<Alert>();
    }

    private async Task<string> ExecuteScriptAsync(string script, string args)
    {
        ProcessStartInfo psi = new()
        {
            FileName = _settings.PythonPath,
            Arguments = $"{script} {args}",
            RedirectStandardOutput = true,
            RedirectStandardError = true,
            UseShellExecute = false,
            CreateNoWindow = true,
            WorkingDirectory = _settings.WorkingDirectory
        };

        using Process process = new() { StartInfo = psi };
        process.Start();

        string output = await process.StandardOutput.ReadToEndAsync();
        await process.WaitForExitAsync();

        if (process.ExitCode != 0)
        {
            string error = await process.StandardError.ReadToEndAsync();
            throw new InvalidOperationException($"Python script failed: {error}");
        }

        return output;
    }
}
```

### Services

#### Alert Service

```csharp
public class AlertService : IAlertService
{
    private readonly IAlertRepository _repository;
    private readonly IPythonBridge _pythonBridge;
    private readonly ILogger<AlertService> _logger;

    public async Task<int> FetchAndStoreAlertsAsync()
    {
        IEnumerable<Alert> alerts = await _pythonBridge.FetchAlertsAsync();

        foreach (Alert alert in alerts)
        {
            await _repository.AddAsync(alert);
        }

        await _repository.SaveChangesAsync();
        _logger.LogInformation("Stored {Count} alerts", alerts.Count());

        return alerts.Count();
    }
}
```

#### Background Scanner

```csharp
public class AlertScannerBackgroundService : BackgroundService
{
    private readonly IServiceScopeFactory _scopeFactory;
    private readonly ILogger<AlertScannerBackgroundService> _logger;
    private readonly TimeSpan _scanInterval = TimeSpan.FromMinutes(2);

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            if (IsMarketOpen())
            {
                using IServiceScope scope = _scopeFactory.CreateScope();
                IAlertService alertService = scope.ServiceProvider.GetRequiredService<IAlertService>();

                try
                {
                    int count = await alertService.FetchAndStoreAlertsAsync();
                    _logger.LogInformation("Scan complete: {Count} alerts", count);
                }
                catch (Exception ex)
                {
                    _logger.LogError(ex, "Scan failed");
                }
            }

            await Task.Delay(_scanInterval, stoppingToken);
        }
    }

    private bool IsMarketOpen()
    {
        // US market hours: 9:30 AM - 4:00 PM ET
        TimeZoneInfo eastern = TimeZoneInfo.FindSystemTimeZoneById("America/New_York");
        DateTime now = TimeZoneInfo.ConvertTimeFromUtc(DateTime.UtcNow, eastern);

        TimeSpan marketOpen = new(9, 30, 0);
        TimeSpan marketClose = new(16, 0, 0);

        return now.TimeOfDay >= marketOpen && now.TimeOfDay <= marketClose
            && now.DayOfWeek != DayOfWeek.Saturday
            && now.DayOfWeek != DayOfWeek.Sunday;
    }
}
```

---

## Blazor UI

### Layout

```razor
@* MainLayout.razor *@
@inherits LayoutComponentBase

<MudThemeProvider />
<MudDialogProvider />
<MudSnackbarProvider />

<MudLayout>
    <MudAppBar Elevation="1">
        <MudIconButton Icon="@Icons.Material.Filled.Menu"
                       Color="Color.Inherit"
                       Edge="Edge.Start"
                       OnClick="@ToggleDrawer" />
        <MudText Typo="Typo.h6">Momentum Screener</MudText>
        <MudSpacer />
        <MudText Typo="Typo.body2">@_marketStatus</MudText>
    </MudAppBar>

    <MudDrawer @bind-Open="_drawerOpen" Elevation="1">
        <NavMenu />
    </MudDrawer>

    <MudMainContent>
        <MudContainer MaxWidth="MaxWidth.ExtraLarge" Class="mt-4">
            @Body
        </MudContainer>
    </MudMainContent>
</MudLayout>
```

### Dashboard Page

```razor
@* Pages/Dashboard.razor *@
@page "/"
@inject IAlertService AlertService
@inject IPositionService PositionService

<PageTitle>Dashboard</PageTitle>

<MudGrid>
    <MudItem xs="12" md="4">
        <MudPaper Class="pa-4">
            <MudText Typo="Typo.h6">Market Condition</MudText>
            <MudText Typo="Typo.h3" Color="@GetConditionColor()">
                @_marketCondition.Score/100
            </MudText>
            <MudText>@_marketCondition.Category</MudText>
        </MudPaper>
    </MudItem>

    <MudItem xs="12" md="4">
        <MudPaper Class="pa-4">
            <MudText Typo="Typo.h6">Today's P&L</MudText>
            <MudText Typo="Typo.h3" Color="@GetPnlColor()">
                @_todayPnl.ToString("C")
            </MudText>
        </MudPaper>
    </MudItem>

    <MudItem xs="12" md="4">
        <MudPaper Class="pa-4">
            <MudText Typo="Typo.h6">Open Positions</MudText>
            <MudText Typo="Typo.h3">@_openPositions.Count</MudText>
        </MudPaper>
    </MudItem>

    <MudItem xs="12">
        <MudPaper Class="pa-4">
            <MudText Typo="Typo.h6">Recent Alerts</MudText>
            <MudTable Items="@_recentAlerts" Dense="true">
                <HeaderContent>
                    <MudTh>Time</MudTh>
                    <MudTh>Ticker</MudTh>
                    <MudTh>Type</MudTh>
                    <MudTh>Price</MudTh>
                    <MudTh>Change</MudTh>
                </HeaderContent>
                <RowTemplate>
                    <MudTd>@context.Timestamp.ToString("HH:mm")</MudTd>
                    <MudTd>@context.Ticker</MudTd>
                    <MudTd>@context.AlertType</MudTd>
                    <MudTd>@context.Price.ToString("C")</MudTd>
                    <MudTd>@context.ChangePercent.ToString("P1")</MudTd>
                </RowTemplate>
            </MudTable>
        </MudPaper>
    </MudItem>
</MudGrid>
```

---

## Python Tools

### Directory Structure

```
tools/py/
├── screener/
│   ├── __init__.py
│   ├── fetch_alerts.py      # Main alert fetcher
│   └── tradingview_client.py
├── data/
│   ├── __init__.py
│   └── market_data.py       # yfinance wrapper
└── analysis/
    ├── __init__.py
    └── performance.py       # Analysis scripts
```

### Alert Fetcher

```python
# tools/py/screener/fetch_alerts.py
import json
import sys
from datetime import datetime
from tradingview_screener import Query

def fetch_alerts():
    """Fetch momentum alerts from TradingView screener."""
    query = (Query()
        .select('name', 'close', 'volume', 'relative_volume_10d_calc',
                'change', 'change_from_open', 'sector')
        .where(Query.Column('close') < 20)
        .order_by('relative_volume_10d_calc', ascending=False)
        .limit(100))

    results = query.get_scanner_data()

    alerts = []
    for _, row in results[1].iterrows():
        alert = {
            'ticker': row['name'],
            'price': float(row['close']),
            'volume': int(row['volume']),
            'relativeVolume': float(row['relative_volume_10d_calc']),
            'changePercent': float(row['change']),
            'changeFromOpen': float(row.get('change_from_open', 0)),
            'sector': row.get('sector', 'Unknown'),
            'timestamp': datetime.utcnow().isoformat(),
            'alertType': classify_alert(row)
        }
        alerts.append(alert)

    return alerts

def classify_alert(row):
    """Classify alert type based on characteristics."""
    change = float(row['change'])
    rel_vol = float(row['relative_volume_10d_calc'])

    if change >= 15:
        return 'PriceSpike'
    elif rel_vol >= 5:
        return 'VolumeClimber'
    else:
        return 'VolumeNewcomer'

if __name__ == '__main__':
    if '--json' in sys.argv:
        alerts = fetch_alerts()
        print(json.dumps(alerts))
    else:
        alerts = fetch_alerts()
        for alert in alerts:
            print(f"{alert['ticker']}: {alert['changePercent']:.1f}%")
```

---

## Configuration

### appsettings.json

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Data Source=data/momentum.db"
  },
  "Python": {
    "PythonPath": ".venv/bin/python",
    "WorkingDirectory": "."
  },
  "Telegram": {
    "BotToken": "",
    "ChatId": ""
  },
  "Trading": {
    "CurrentPhase": "PaperTrading",
    "ScanIntervalMinutes": 2
  }
}
```

### Program.cs

```csharp
WebApplicationBuilder builder = WebApplication.CreateBuilder(args);

// Add services
builder.Services.AddRazorComponents()
    .AddInteractiveServerComponents();
builder.Services.AddMudServices();

// Database
builder.Services.AddDbContext<AppDbContext>();

// Repositories
builder.Services.AddScoped<IAlertRepository, AlertRepository>();
builder.Services.AddScoped<IPositionRepository, PositionRepository>();
builder.Services.AddScoped<ITradeRepository, TradeRepository>();

// Services
builder.Services.AddScoped<IAlertService, AlertService>();
builder.Services.AddScoped<ITradingEngine, TradingEngine>();
builder.Services.AddScoped<IRiskManager, RiskManager>();
builder.Services.AddSingleton<IPythonBridge, PythonBridge>();

// Background services
builder.Services.AddHostedService<AlertScannerBackgroundService>();

// Configuration
builder.Services.Configure<PythonSettings>(builder.Configuration.GetSection("Python"));
builder.Services.Configure<TelegramSettings>(builder.Configuration.GetSection("Telegram"));

WebApplication app = builder.Build();

app.UseStaticFiles();
app.UseAntiforgery();

app.MapRazorComponents<App>()
    .AddInteractiveServerRenderMode();

app.Run();
```

---

## Testing

### Unit Test Example

```csharp
public class RiskManagerTests
{
    [Fact]
    public void CalculatePositionSize_WithValidInputs_ReturnsCorrectSize()
    {
        // Arrange
        RiskManager manager = new();
        Alert alert = new() { Price = 10.00m };
        RiskProfile profile = new() { MaxRiskPerTrade = 0.02m };
        decimal accountBalance = 1000m;

        // Act
        decimal positionSize = manager.CalculatePositionSize(alert, profile, accountBalance);

        // Assert
        positionSize.Should().Be(20m); // 2% of 1000
    }
}
```

### Integration Test Example

```csharp
public class AlertRepositoryTests : IDisposable
{
    private readonly AppDbContext _context;
    private readonly AlertRepository _repository;

    public AlertRepositoryTests()
    {
        DbContextOptions<AppDbContext> options = new DbContextOptionsBuilder<AppDbContext>()
            .UseInMemoryDatabase(databaseName: Guid.NewGuid().ToString())
            .Options;

        _context = new AppDbContext(options);
        _repository = new AlertRepository(_context);
    }

    [Fact]
    public async Task AddAsync_WithValidAlert_PersistsToDatabase()
    {
        // Arrange
        Alert alert = new()
        {
            Id = Guid.NewGuid(),
            Ticker = "TEST",
            Price = 10.00m,
            Timestamp = DateTime.UtcNow
        };

        // Act
        await _repository.AddAsync(alert);
        await _repository.SaveChangesAsync();

        // Assert
        Alert? retrieved = await _repository.GetByIdAsync(alert.Id);
        retrieved.Should().NotBeNull();
        retrieved!.Ticker.Should().Be("TEST");
    }

    public void Dispose() => _context.Dispose();
}
```
