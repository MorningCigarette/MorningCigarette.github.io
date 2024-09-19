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
 public partial class App : Application
 {
     public IServiceProvider ServiceProvider { get; }

     public new static App Current => (App)Application.Current;
     public App()
     {
         ServiceProvider = ConfigureServices();
     }


     private static IServiceProvider ConfigureServices()
     {
         var services = new ServiceCollection();
         services.AddSingleton<MainWindow>();
         services.AddSingleton<MainWindowViewModel>();
         services.AddDbContext<DefaultDbContext>();
         services.AddSingleton<ILogger>(_ =>
         {
             return new LoggerConfiguration()
                   .MinimumLevel.Debug()
                   .WriteTo.File("Log/log.txt", rollingInterval: RollingInterval.Day)
                   .CreateLogger();
         });


         return services.BuildServiceProvider();
     }

     private void Application_Startup(object sender, StartupEventArgs e)
     {
         var mainWindow = ServiceProvider.GetRequiredService<MainWindow>();
         mainWindow.Show();
     }
 }
```

```xaml
<Application
    x:Class="Wpf_onlyDI.App"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="clr-namespace:Wpf_onlyDI" Startup="Application_Startup">
    <Application.Resources />
</Application>
```

```csharp
public partial class MainWindow : Window
{
    public MainWindow(MainWindowViewModel viewModel)
    {
        InitializeComponent();
        this.DataContext = viewModel;
    }
}
```

```csharp


public partial class MainWindowViewModel : ObservableObject
{
    private readonly ILogger _logger;
    [ObservableProperty]
    private string title = "hello";

    public MainWindowViewModel(ILogger logger)
    {
        this._logger = logger;

        _logger.Information(title);
    }
}
```



第二种方式（基于 .NET Generic Host,仅仅.NET Core）：

```csharp
   public partial class App : Application
   {
       private static readonly IHost _host = Host.CreateDefaultBuilder()
           .ConfigureAppConfiguration(c =>
           {
               c.SetBasePath(AppContext.BaseDirectory);
           })
           .ConfigureServices(services =>
           {
               services.AddHostedService<ApplicationHostService>();
               services.AddSingleton<MainWindow>();
               services.AddSingleton<MainWindowViewModel>();
               services.AddDbContext<DefaultDbContext>();
           })
           .ConfigureLogging(logging =>
           {
               logging.ClearProviders();
               Log.Logger = new LoggerConfiguration()
               .WriteTo.File("Log/log.txt", rollingInterval: RollingInterval.Day)
               .CreateLogger();
               logging.AddSerilog(Log.Logger);
           })
           .Build();



       private void OnStartup(object sender, StartupEventArgs e)
       {
           _host.Start();
       }

       private void OnExit(object sender, ExitEventArgs e)
       {
           _host.StopAsync().Wait();
           _host.Dispose();
       }

       public static T GetRequiredService<T>() where T : class
       {
           return _host.Services.GetRequiredService<T>();

       }
   }
```

```xaml
<Application
    x:Class="Wpf_Generic_Host.App"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="clr-namespace:Wpf_Generic_Host"
    Startup="OnStartup"
    Exit="OnExit"
    >
    <Application.Resources />
</Application>
```

```csharp
internal class ApplicationHostService : IHostedService
{
    private readonly IServiceProvider _serviceProvider;

    public ApplicationHostService(IServiceProvider serviceProvider)
    {
        this._serviceProvider = serviceProvider;
    }
    public async Task StartAsync(CancellationToken cancellationToken)
    {
        await HandleActivationAsync();
    }

    public Task StopAsync(CancellationToken cancellationToken)
    {
        return Task.CompletedTask;
    }

    /// <summary>
    /// Creates main window during activation.
    /// </summary>
    /// <returns></returns>
    private Task HandleActivationAsync()
    {
        if (Application.Current.Windows.OfType<MainWindow>().Any())
        {

            return Task.CompletedTask;
        }
        var mainWindow = _serviceProvider.GetRequiredService<MainWindow>();
        mainWindow.Loaded += MainWindow_Loaded;
        mainWindow?.Show();
        return Task.CompletedTask;

    }

    private void MainWindow_Loaded(object sender, RoutedEventArgs e)
    {
        if (sender is not MainWindow mainWindow)
        {
            return;
        }

        //_ = mainWindow.NavigationView.Navigate(typeof(DashboardPage));
    }
}
```
