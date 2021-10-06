---
layout: post
title:  "Kap 9 - 2021-10-04 Monitorering av moln applikationer"
date:   2021-10-04 09:31:04 +0200
categories: jekyll update
image1: /Image/Monitoring/InsightChart.png
image2: /Image/Monitoring/Traces.png
---

# Beskriv kort applikationen, vad gör den?

Det jag har lagt till i min app är hur man monitorera processen. Då ser man från starten där programmet körs och har också lagt
ett extra loginfo när man trycker på Overview tabben i min razor pages där det står (Items has been added). Därefer har
publishat detta till min azure web service där man kan se all log traces.


# Gör en diagram (använd eg draw.io) hur den applikation är förbundet andra tjäster

![Flowchart]({{ page.image1 }})

# Beskriv koden, med fokus på den del som rör din log implementation

- Laddade ner följande Nuget:
    1. Microsoft.ApplicationInsights
    2. Microsoft.ApplicationInsights.AspNetCore
    3. Serlilog.AspNetCore
    4. Serilog.Settings.Configuration
    5. Serilog.Sinks.ApplicationInsights
    6. Serilog.Sinks.File

- Som ni ser på program.cs filen så har jag först och främst lagt till en try n catch log hantering
    och detta är för att säkerställa att konfigurations issues är loggad. UseSerilog metoden som ni ser i CreateHostBuilder metoden
    anropar ConfigureServices på IHostBuilder och lägger till en instans av SerilogLoggerFactory som programmets ILoggerFactory.
    När en ILoggerFactory krävs av appen (för att skapa en ILogger) kommer SerilogLoggerFactory att användas.

``` csharp

public class Program
    {
        public static int Main(string[] args)
        {
            Log.Logger = new LoggerConfiguration()
            .MinimumLevel.Override("Microsoft", LogEventLevel.Information)
            .Enrich.FromLogContext()
            .WriteTo.Console()
            .WriteTo.ApplicationInsights(new TelemetryConfiguration { InstrumentationKey = "INSTRUMENTKEY" }, TelemetryConverter.Traces)
            .CreateLogger();
            try
            {
                Log.Information("Starting web host");
                CreateHostBuilder(args).Build().Run();
                return 0;
            }
            catch (Exception e)
            {
                Log.Fatal(e, "Host terminated unexpectedly");
                return 1;
            }
            finally
            {
                Log.CloseAndFlush();
            }
        }
        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
            .UseSerilog()
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>();
                });
        }


```

- Här nedan ser ni min JSON innehåll:
    Här är all innehåll för min Serilog och jag har t.o.m gjort så att jag kan spara allt i en txt fil.
    InstrumentationKey är det som jag har hämtat från min application insight app med resterande innehåll.

``` json

{
  "Serilog": {
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Warning",
        "Microsoft.Hosting.Lifetime": "Information"
      }
    },
    "WriteTo": [
      {
        "Name": "File",
        "Args": {
          "path": "./logs/log-.txt",
          "rollingInterval": "Day"
        }
      }
    ]
  },
  "AllowedHosts": "*",
  "InstrumentationKey": "INSTRUMENTKEY",
  "CosmosDb": {
    "Account": "https://clothes-account.documents.azure.com:443/",
    "Key": "cVFvk9QnAmZC3Eht5cvQNq4X0yu1GpMoIVsVaOlKireOLYyUAlU5lqib2izDK1mOlPwVjUgSkoEDvh81v6YnkQ==",
    "DatabaseName": "Clothes",
    "ContainerName": "my-container"
  },
  "ApplicationInsights": {
    "ConnectionString": "INSTRUMENTKEY"
  }
}

```

- Här är DI i min startup.cs fil och dessa har jag använt i min DI.
    UseSerilogRequestLogging middlewaren kondenserar viktig information om requests hanteras
    av ASP.NET Core till en ren, streamlined, request om slutförande.

``` csharp

public void ConfigureServices(IServiceCollection services)
        {
            services.AddRazorPages();
            services.AddSingleton<ICosmosDbService>(InitializeCosmosClientInstanceAsync(Configuration.GetSection("CosmosDb")).GetAwaiter().GetResult());
            services.AddSingleton(Log.Logger);
            services.AddApplicationInsightsTelemetry("InstrumentationKey");
            //services.AddApplicationInsightsTelemetry(Configuration["APPINSIGHTS_CONNECTIONSTRING"]);
        }

// Middleware

app.UseSerilogRequestLogging();

```

- Använde logger DI och via det gjorde jag en simpel if-else statment där om det inte finns något item i min page
    så får jag (No items has been found!) men om det finns något så får jag ("Items has been added").

``` csharp

public class ClothesOverviewModel : PageModel
    {
        private readonly ICosmosDbService _cosmosDbService;
        private readonly ILogger<ClothesOverviewModel> _logger;
        public ClothesOverviewModel(ICosmosDbService cosmosDbService, ILogger<ClothesOverviewModel> logger)
        {
            _cosmosDbService = cosmosDbService;
            _logger = logger;
        }

        [BindProperty]
        public List<ClothesWebModel> ClothesList { get; set; } = new List<ClothesWebModel>();
        [BindProperty]
        public ClothesWebModel Clothes { get; set; }

        public async Task<IActionResult> OnGet()
        {
            var result = await _cosmosDbService.GetItemsAsync("SELECT * FROM c");
            if (result == null)
            {
                _logger.LogWarning("No items has been found!");
            }
            else
            {
                foreach (var clothItem in result)
                {
                    ClothesList.Add(clothItem);
                }
                _logger.LogInformation("Items has been added");
            }
            return Page();
        }
    }

```



Så här ser processen ut när man startar programmet och trycker på Overview tabben i consolen:

``` cmd
[17:24:47 INF] Starting web host
[17:24:51 INF] User profile is available. Using 'C:\Users\Kioma\AppData\Local\ASP.NET\DataProtection-Keys' as key repository and Windows DPAPI to encrypt keys at rest.
[17:24:52 INF] Now listening on: https://localhost:5001
[17:24:52 INF] Now listening on: http://localhost:5000
[17:24:52 INF] Application started. Press Ctrl+C to shut down.
[17:24:52 INF] Hosting environment: Development
[17:24:52 INF] Content root path: C:\Users\Kioma\Documents\GitHub\Teknikhögskolan\CosmosClothes\CosmosWebApp
[17:24:53 INF] Request starting HTTP/2 GET https://localhost:5001/ - -
[17:24:53 INF] Executing endpoint '/Index'
[17:24:53 INF] Route matched with {page = "/Index"}. Executing page /Index
[17:24:53 INF] Executing handler method CosmosWebApp.Pages.IndexModel.OnGet - ModelState is Valid
[17:24:53 INF] Executed handler method OnGet, returned result .
[17:24:53 INF] Executing an implicit handler method - ModelState is Valid
[17:24:53 INF] Executed an implicit handler method, returned result Microsoft.AspNetCore.Mvc.RazorPages.PageResult.
[17:24:53 INF] Executed page /Index in 187.6288ms
[17:24:53 INF] Executed endpoint '/Index'
[17:24:54 INF] HTTP GET / responded 200 in 340.7144 ms
[17:24:54 INF] Request finished HTTP/2 GET https://localhost:5001/ - - - 200 - text/html;+charset=utf-8 379.8975ms
[17:25:24 INF] Request starting HTTP/2 GET https://localhost:5001/ClothesOverview - -
[17:25:24 INF] Executing endpoint '/ClothesOverview'
[17:25:25 INF] Route matched with {page = "/ClothesOverview"}. Executing page /ClothesOverview
[17:25:25 INF] Executing handler method CosmosWebApp.Pages.ClothesOverviewModel.OnGet - ModelState is Valid
[17:25:26 INF] Items has been added
[17:25:26 INF] Executed handler method OnGet, returned result Microsoft.AspNetCore.Mvc.RazorPages.PageResult.
[17:25:26 INF] Executed page /ClothesOverview in 1311.5607ms
[17:25:26 INF] Executed endpoint '/ClothesOverview'
[17:25:26 INF] HTTP GET /ClothesOverview responded 200 in 1366.3611 ms
[17:25:26 INF] Request finished HTTP/2 GET https://localhost:5001/ClothesOverview - - - 200 - text/html;+charset=utf-8 1378.3223ms
```

- Därefter publishar jag dessa kod
- Nu vill se mina traces via min application insight och det vi genom att hitta det i vår azure web service och därefter välja Logs tabben
- Så här ser det ut när jag dem 100 st traces via min Kusto query:
    ![Traces]({{ page.image2 }})


# Fundera på och beskriv hur logging av din applikation kan avhjälpa säkerhetsproblem i din applikation

Det jag kan tänka är hjälpsamt med logging är att man kan traca alla steg och då ser man i så fall om
det skulle ske någonslags intrång.

# Förklara dina queries; vad gör dom? varför är dessa data som tas fram interesanta?

Hämtar mina 10 descending traces table.

``` csharp
traces
| order by timestamp desc
| take 10
```
Räknar min traces som är 192 stycken.

``` csharp
traces
| count
```


Referens:
- https://www.youtube.com/watch?v=dI8-UHhiZZE&ab_channel=MeetKamalToday.
- https://github.com/serilog/serilog-aspnetcore
- https://andrewlock.net/adding-serilog-to-the-asp-net-core-generic-host/
- https://nblumhardt.com/2019/10/serilog-mvc-logging/