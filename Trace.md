# Simple trace
Debug messages for the _MqttClient_ and _MqttServer_ can be consumed using a static event. The following code shows how to manually write all trace messages to the console window.

```csharp
// Write all trace messages to the console window.
MqttNetTrace.TraceMessagePublished += (s, e) =>
{
    Console.WriteLine($">> [{e.TraceMessage.Timestamp:O}] [{e.TraceMessage.ThreadId}] [{e.TraceMessage.Source}] [{e.TraceMessage.Level}]: {e.TraceMessage.Message}");
    if (e.TraceMessage.Exception != null)
    {
        Console.WriteLine(e.TraceMessage.Exception);
    }
};
```

# Separated trace
For use cases where multiple clients are running in the same process and thus trace messages must be separated from each other a custom _LogId_ can be specified at the _MqttClientOptions_. The specified ID is used for every trace message which is related to a certain client. This ID is _not_ the _ClientId_. If the _LogId_ is not set the _ClientId_ will be used as the default. The following code shows how to set the _LogId_.

```csharp
// Use a custom identifier for the trace messages.
var clientOptions = new MqttClientOptionsBuilder()
    .WithLogId("ClientX")
    .Build();
```

The information of the used _LogId_ is present on the _Scope_ property of a _LogMessage_. More information about log scopes is available at <https://docs.microsoft.com/en-us/aspnet/core/fundamentals/logging?tabs=aspnetcore2x>.

# Advanced trace
This library uses the library _Microsoft.Extensions.Logging_ for internal logging (https://docs.microsoft.com/en-us/aspnet/core/fundamentals/logging?tabs=aspnetcore2x). 

In order to access the internal logging components it is required to build the service provider manually and make use of the _ILoggerFactory_. The following code shows how to add the _console logger_ from the logging library to the internal logger.
```csharp
var services = new ServiceCollection()
    .AddMqttClient()
    .BuildServiceProvider();

services.GetRequiredService<ILoggerFactory>()
    .AddConsole();
```
The above code only uses the MQTT client and thus only _AddMqttClient_ is called. For server purposes _AddMqttServer_ must be added instead. The method _AddConsole_ requires that the nuget package _Microsoft.Extensions.Logging.Console_ is referenced by the calling project.