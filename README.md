# Build a Banking App using Blazor Server App in .NET 9.0

**Blazor Server vs Blazor WebAssembly**

https://mobidev.biz/blog/blazing-a-trail-web-app-development-with-microsoft-blazor

![Blazor Server Architecture](https://github.com/user-attachments/assets/85a356ca-1bd4-41d0-9944-8ed1c92f712f)

# 1. Set Up the Development Environment

## Install Prerequisites:

+ .NET 9.0 SDK: Download from the .NET website.
+ IDE: Use Visual Studio 2022 with the ASP.NET and web development workload installed.
+ Database: Install SQL Server, PostgreSQL, or your preferred RDBMS.

## Create a Blazor Server App:

Open a terminal or Visual Studio and create a new Blazor Server App:

```
dotnet new blazorserver -o BankingApp
cd BankingApp
```

# 2. Define the App Requirements

## A banking app typically includes:
+ Authentication: User login, role-based access (e.g., customer, admin).
+ Account Management: View balances, account details.
+ Transactions: Fund transfers, transaction history.
+ Admin Panel: Manage users, accounts, and settings.

# 3. Structure the Application

## Organize the project into logical layers:
+ Models: Define data structures (e.g., Account, Transaction, User).
+ Services: Handle business logic (e.g., transaction processing).
+ Data Access Layer: Use Entity Framework Core (EF Core) for database operations.
+ UI Components: Build Razor components for the user interface.

# 4. Set Up the Database

## Add EF Core to the Project: Install EF Core NuGet packages:
```
dotnet add package Microsoft.EntityFrameworkCore
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
dotnet add package Microsoft.Extensions.Configuration
```

## Multiple environments configuration files
+ appsettings.Development.json
+ appsettings.Staging.json
+ appsettings.Production.json

For Example (appsettings.Production.json):

```
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=BankingApp;Trusted_Connection=True;"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "EnvironmentSpecificSetting": "DevelopmentSetting"
}
```

## Program.cs
```
var builder = WebApplication.CreateBuilder(args);

// Add services to the container
builder.Configuration.AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
    .AddJsonFile($"appsettings.{builder.Environment.EnvironmentName}.json", optional: true)
    .AddEnvironmentVariables();

var app = builder.Build();
```

## Create the Database Context: Define a BankingDbContext class to interact with the database:
```
using Microsoft.EntityFrameworkCore;

public class BankingDbContext : DbContext
{
    public BankingDbContext(DbContextOptions<BankingDbContext> options) : base(options) { }

    public DbSet<Account> Accounts { get; set; }
    public DbSet<Transaction> Transactions { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer("YourConnectionString");
    }
}
```

## Run Migrations: Create and apply migrations:
```
dotnet ef migrations add InitialCreate
dotnet ef database update
```

# 5. Implement Authentication

## Add Identity for Authentication: Add ASP.NET Core Identity to the project:
```
dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
```

## Configure And Register Identity services:
```
builder.Services.AddIdentity<ApplicationUser, IdentityRole>()
    .AddEntityFrameworkStores<BankingDbContext>();

builder.Services.AddDefaultIdentity<IdentityUser>()
    .AddRoles<IdentityRole>()
    .AddEntityFrameworkStores<BankingDbContext>();
```

## Register DbContext in Dependency Injection:
```
builder.Services.AddDbContext<BankingDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
```

## Protect Pages: Use the [Authorize] attribute to restrict access:
```
@attribute [Authorize]
```

# 6. Build the User Interface

## Create Razor Components:
+ Dashboard: Dashboard.razor
+ Account Details: AccountDetails.razor
+ Transaction History: TransactionHistory.razor
+ Fund Transfer: TransferFunds.razor

Example for viewing account details:

```
@page "/account/{accountId}"
@inject BankingService BankingService

<h3>Account Details</h3>
<div>
    @if (account != null)
    {
        <p>Account Number: @account.Number</p>
        <p>Balance: @account.Balance</p>
    }
</div>

@code {
    [Parameter]
    public int AccountId { get; set; }
    private Account account;

    protected override async Task OnInitializedAsync()
    {
        account = await BankingService.GetAccountById(AccountId);
    }
}
```

## Use Bootstrap: Add Bootstrap for responsive and clean styling.

# 7. Implement Business Logic

## Define Services: Create services for handling business logic, e.g., BankingService:
```
public class BankingService
{
    private readonly BankingDbContext _context;

    public BankingService(BankingDbContext context)
    {
        _context = context;
    }

    public async Task<Account> GetAccountById(int id)
    {
        return await _context.Accounts.FindAsync(id);
    }

    public async Task<bool> TransferFunds(int fromAccountId, int toAccountId, decimal amount)
    {
        // Add transfer logic here
        return true;
    }
}
```

## Register Services: In Program.cs, register the service:
```
builder.Services.AddScoped<BankingService>();
```

## Monitor and Maintain

### Logging and Monitoring:
+ Use tools like Serilog or Application Insights for logging.
+ Monitor performance, errors, and usage analytics.

### Regular Updates:
Keep dependencies and frameworks up-to-date to ensure security and performance.

# 8. Test and Debug
+ Run the application locally using dotnet run or the IDE's debug mode.
+ Use Postman or Swagger for API testing (if needed).

## Testing for Multiple Environments
+ Development: Test with local database and mock services.
+ Staging: Test in a near-production environment with production-like configurations.
+ Production: Monitor and test in production with real user data using Application Insights or similar tools.

# 9. Secure the Application

## Data Protection:
+ Use HTTPS for all communications.
+ Encrypt sensitive data (e.g., passwords, transactions).

## Authentication
+ Implement JWT Tokens or ASP.NET Core Identity for secure login.

## Input Validation
+ Validate all user inputs to prevent SQL injection and XSS attacks.

# 10. Deploy the Application

## Publish the App: Run the following command:
```
dotnet publish -c Release -o publish
```

## Host the App:
+ Use Azure App Service, AWS, or on-premise IIS for hosting.
+ Configure the database connection string for the production environment.
