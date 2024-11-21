# Logging

## Unittest

FakeLogger (.NET 8): <https://stackoverflow.com/questions/43424095/how-to-unit-test-with-ilogger-in-asp-net-core/78414479#78414479>

## Static Logger

- <https://levelup.gitconnected.com/how-to-eliminate-injecting-ilogger-t-everywhere-in-net-301be7b367a8>

```cs
// todo verify
var builder = WebApplication.CreateBuilder();
var serviceProvider = builder.Services.BuildServiceProvider();
StaticLoggerFactory.Initialize(serviceProvider.GetRequiredService<ILoggerFactory>());
```

simple:

```cs
ILogger logger = context.ServiceProvider.GetRequiredService<ILogger<Program>>();
```

from Hangfire:

```cs
private static readonly ILog Logger = LogProvider.GetCurrentClassLogger();
```

## Structured Logging

<https://serilog.net/>

## OpenTelemetry (otel)

From <https://learn.microsoft.com/en-us/dotnet/core/diagnostics/observability-with-otel>

```cmd
dotnet new webapi -o aspnetacorewebapi

cd aspnetacorewebapi

dotnet add package OpenTelemetry.Exporter.Console
dotnet add package OpenTelemetry.Extensions.Hosting
dotnet add package OpenTelemetry.Instrumentation.AspNetCore

rem needed for /metrics endpoint (needed for prometheus): 
OpenTelemetry.Exporter.Prometheus.AspNetCore
```

Service config without prometeus endpoint:

```cs
// Configure OpenTelemetry with tracing and auto-start.
builder.Services
    .AddOpenTelemetry()
        .ConfigureResource(resource => resource
            .AddService(serviceName: builder.Environment.ApplicationName))
        .WithTracing(tracing => tracing
            .AddAspNetCoreInstrumentation()
            .AddConsoleExporter());
```

Service config with prometheus endpoint (/metrics):

```cs
// Configure OpenTelemetry with tracing and auto-start.
builder.Services
    .AddOpenTelemetry()
        .ConfigureResource(resource => resource
            .AddService(serviceName: builder.Environment.ApplicationName))
        .WithTracing(tracing => tracing
            .AddAspNetCoreInstrumentation()
            .AddConsoleExporter())
        .WithMetrics(metrics => metrics
            .AddAspNetCoreInstrumentation() // add some basic log data to /metrics endpoint
            .AddPrometheusExporter() // configure /metrics endpoint
        );
```

Middleware:

```cs
// Configure the Prometheus scraping endpoint
app.MapPrometheusScrapingEndpoint();
```

Run App, call <url>/metrics
