# Blog der 5. Einheit

## Zusammenfassung vom 26.11.2021

Matthias Mandl und Peter Vadle

## Bond

[Bond GitHub page](https://github.com/microsoft/bond)

Bond ist ein plattformübergreifendes Open-Source-Framework für die Arbeit von schematisierten Daten.
Bond verwendet binäre Kommunikation. 
Bond unterstützt viele Sprachen, wie C++, C#, Java, Python und kann auch statt Protobuf für gRPC verwendet werden.
Damit können Daten binär serialisiert und übertragen werden, wodurch (teilweise/situationsabhängig) weniger Datenmengen versendet werden müssen.
Z.B. für Azure Background Services kann dies nützlich sein, da hier für die Kommunikation gezahlt wird und weniger gezahlt werden muss, wenn weniger Daten gesendet werden müssen.

## Logging

Seit .Net Core gibt es ILogger und ILoggerFactory für verschiedene Provider.

Durch Host.CreateDefaultBuilder werden einige Provider automatisch konfiguriert.
[CreateDefaultBuilder implementation](https://source.dot.net/#Microsoft.Extensions.Hosting/Host.cs,f78c881a376d7fa4)
[Defaults Implementation](https://source.dot.net/#Microsoft.Extensions.Hosting/HostingHostBuilderExtensions.cs,5d86d5acb42aaed3)
Host.CreateDefaultBuilder ermöglicht automatisch (im Bezug auf logging):
* falls Windows, wird ein logging-Filter eingetragen, für den EventLogLoggerProvider, welcher in den Windows-Event-View (=Event tracing für Windows) loggt.
* falls Windows => logging.AddEventlog() für das loggen in die Windows-Event-View
* falls nicht im Browser => logging.AddConsole()
* logging.AddDebug() (debug fenster in VS)
* logging.AddEventSourceLogger() (definierte Datei in Windows und Linux)
* logging.AddConfiguration(hostingContext.Configuration.GetSection("Logging")) (für eigenes Konfigurieren von Logging)

Angeben eines Loglevels => es wird für das angegebene Level oder höher geloggt.
logging.ClearProvider() => default einstellungen werden gelöscht.
logging.AddApplicationInsights(<Application Insights Key>) => Application Insights von Azure benutzen.

File loggen => z.B. mit Serilog Nuget Package:  
```csharp
builder.AddConfiguration(configuration.GetSection("Logging"))
.AddSerilog(new LoggerConfiguration().writeTo.File("logFile.txt").CreateLogger())
```

in der Klasse kann nun der ILogger<Klasse> injected werden und dort verwendet werden mit z.B. (Extensions .LogError .LogDebug ersparrt LogLevel angeben)
```csharp
_logger.Log(LogLevel.Debug, "test logging")
```
Anstatt ILogger kann auch die ILoggerFactory injected werden und damit ein Logger erstellt werden

Activity 
für z.b. wenn ich eine Methode aufrufe, die eine Methode aufruft und ich wissen möchte zu welcher methode was dazu gehört, können die Logs durch Activitys hierarchisch ausgeben werden.
ActivitySource schreibt in einen static member eine activity ID hinein sobald eine Activity gestartet wird.
die Log Methoden, wie z.b.
_logger.LogError verwendet diese ActivityID und ein Tool welches dies unterstützt kann diese dann hierachisch ausgeben.


## Event Counter Api

Man kann auch eigene Counts mit der
```csharp
Eventsource
```
Klasse schreiben:

OnEventCommand der Basisklasse wird überschrieben und wenn jemand zuhört werden Counters erstellt z.b. 
```csharp
IncrementingEventCounter 
PollingCounter
```

## Open Telemetry

[Open Telemetry Seite](https://opentelemetry.io/)

Standard für Telemetrie Daten wie Traces, Metrics und Logs

Nuget Packages OpenTelemetry + OpenTelemetry.Exporter.Console (=exporter für Console)

Ohne Dependency Injection: 
```csharp
using var tracerProvider = Sdk.CreateTracerProviderBuilder()
    .SetResourceBuilder(ResourceBuilder.CreateDefault().AddService("OpenTelemetrySample"))
    .AddSource("LoggingSample.DistributedTracing")
    .AddConsoleExporter()
    .Build();
```

Mit Dependency Injection: 
```csharp
logging.AddOpenTelemetry(options => options.AddConsoleExporter())
```



## .Net Tools

dotnet-trace (für logging)
 man kann sich reinhängen in die Prozesse wo man logging collecten kann 

dotnet counters (für monitoring)
 list = stellt liste der counters dar
 ps = gibt die dotnet prozesse an die gemonitored werden können
 collect = monitored in ein file
 monitor = startet monitoring einer .Net Applikation

dotnet dump (für analyse von dumps)
























