---
title: "WPF引用程序中的依赖注入方法比较"
description: 
slug: "Wpf-Injection"
date: 2024-08-06
categories:
    - MVVM

---
第一种方式（基于 Application 类的静态方法）：

```csharp
public sealed partial class App : Application
{
    public App()
    {
        Services = ConfigureServices();
        this.InitializeComponent();
    }

    public new static App Current => (App)Application.Current;

    public IServiceProvider Services { get; }

    private static IServiceProvider ConfigureServices()
    {
        var services = new ServiceCollection();

        services.AddSingleton<IFilesService, FilesService>();
        services.AddSingleton<ISettingsService, SettingsService>();
        services.AddSingleton<IClipboardService, ClipboardService>();
        services.AddSingleton<IShareService, ShareService>();
        services.AddSingleton<IEmailService, EmailService>();

        services.AddDbContext<BoardGameContext>(dbBuilder =>
        {
            dbBuilder.UseSqlite("Data Source=Rentopoly.db");
        });

        return services.BuildServiceProvider();
    }
}
```

第二种方式（基于 .NET Generic Host）：

```csharp
public partial class App : Application
{
    [STAThread]
    public static void Main(string[] args)
    {
        using IHost host = CreateHostBuilder(args).Build();
        host.Start();

        using (var scope = host.Services.GetRequiredService<IServiceScopeFactory>().CreateScope())
        using (BoardGameContext context = scope.ServiceProvider.GetRequiredService<BoardGameContext>())
        {
            context.Database.Migrate();
        }

        App app = new();
        app.InitializeComponent();
        app.MainWindow = host.Services.GetRequiredService<MainWindow>();
        app.MainWindow.Visibility = Visibility.Visible;
        app.Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
        .ConfigureAppConfiguration((hostBuilderContext, configurationBuilder)
            => configurationBuilder.AddUserSecrets(typeof(App).Assembly))
        .ConfigureServices((hostContext, services) =>
        {
            services.AddSingleton<MainWindow>();
            services.AddSingleton<AddRentalViewModel>();
            services.AddSingleton<ViewRentalsViewModel>();
            services.AddSingleton<CatalogViewModel>();

            services.AddSingleton<Func<Rental, RentalItemViewModel>>(serviceProvider =>
            {
                return (Rental rental) => new RentalItemViewModel(rental, 
                        serviceProvider.GetRequiredService<BoardGameContext>(),
                        serviceProvider.GetRequiredService<IMessenger>());
            });

            services.AddSingleton<WeakReferenceMessenger>();
            services.AddSingleton<IMessenger, WeakReferenceMessenger>(provider => provider.GetRequiredService<WeakReferenceMessenger>());

            services.AddSingleton(_ => Current.Dispatcher);

            services.AddDbContext<BoardGameContext>(dbBuilder =>
            {
                dbBuilder.UseSqlite("Data Source=Rentopoly.db");
            });

            services.AddTransient<ISnackbarMessageQueue>(provider =>
            {
                Dispatcher dispatcher = provider.GetRequiredService<Dispatcher>();
                return new SnackbarMessageQueue(TimeSpan.FromSeconds(2.0), dispatcher);
            });
        });
}
```

在比较这两种方式配置依赖注入和服务的方法时，我们可以从几个关键方面进行分析：

### 设计模式和架构：

- **第一种方式**（基于 Application 类的静态方法）:
    
    - 这种方法直接在 `App` 类中初始化服务。它使用 `ServiceCollection` 来注册服务，并创建一个 `IServiceProvider` 实例来解析这些服务。
    - 这种方式较为简单，适合小型应用或者应用的原型开发，但可能难以管理复杂的依赖关系或进行单元测试。
- **第二种方式**（基于 .NET Generic Host）:
    
    - 这种方法使用 `.NET Core` 的 `HostBuilder`，这是一种更现代的依赖注入和应用启动方法。它不仅支持依赖注入，还支持日志、配置和生命周期管理。
    - 这种方式更适合构建大型应用，因为它提供了更强的灵活性和扩展性。通过使用 `HostBuilder`，可以轻松地添加更多中间件和服务。

### 服务配置和管理：

- **第一种方式**:
    
    - 服务直接在应用启动时一次性配置和创建。
    - 由于所有配置都在一个地方，对于简单的应用来说，这种方式可能更易于管理。
- **第二种方式**:
    
    - 服务的生命周期和作用域可以更精细地控制。例如，可以配置某些服务为单例，而其他服务为每次请求创建新的实例。
    - 支持通过配置文件、环境变量等多种方式动态配置服务，这对于需要高度可配置性的大型应用非常重要。

### 应用的启动和运行：

- **第一种方式**:
    
    - 应用的启动较为直接，只涉及到创建 `App` 实例并初始化组件。
    - 这种方式可能在应用启动速度上有优势，因为它不涉及复杂的构建和配置解析过程。
- **第二种方式**:
    
    - 应用通过 `Main` 方法启动，这是 .NET Core 应用的典型启动方式。在这个过程中，应用会构建和配置宿主，然后启动宿主。
    - 虽然这种方式在启动时可能稍慢，因为需要加载和配置宿主，但它提供了更多的灵活性，例如在启动过程中运行数据库迁移。

### 数据库上下文管理：

- **两种方式**:
    - 都使用了 `AddDbContext` 来注册 `BoardGameContext`。这表明无论选择哪种方式，数据库上下文的配置和使用都是类似的。
    - 在第二种方式中，数据库迁移可以在应用启动时自动执行，这是一个额外的优势，因为它简化了数据库管理。

### 总结：

选择哪种方式主要取决于应用的规模、复杂性以及团队对技术栈的熟悉程度。对于小型、简单的应用或快速原型开发，第一种方式可能更合适。而对于需要高度可扩展和可配置的大型应用，第二种方式则可能是更好的选择。
