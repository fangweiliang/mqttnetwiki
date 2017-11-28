# Simple trace
Debug messages for the _MqttClient_ and _MqttServer_ can be consumed using a static event. The following code shows how to manually write all trace messages to the console window.

```csharp
// Write all trace messages to the console window.
MqttNetGlobalLog.LogMessagePublished += (s, e) =>
{
    var trace = $">> [{e.TraceMessage.Timestamp:O}] [{e.TraceMessage.ThreadId}] [{e.TraceMessage.Source}] [{e.TraceMessage.Level}]: {e.TraceMessage.Message}";
    if (e.TraceMessage.Exception != null)
    {
        trace += Environment.NewLine + e.TraceMessage.Exception.ToString();
    }

    Console.WriteLine(trace);
};
```

# Extended trace
For use cases where multiple clients are running in the same process and thus trace messages must be separated between the clients a custom logger must be specified via the constructor of the client and server. 

The built in logger of this project allows specifying a log ID which must be set in the constructor of the logger. Doing this requires using a different constructor. The specified _LogId_ can be accessed in the _MqttNetLogMessage_ class for every generated log message.

The following code shows how to use a custom log ID.

```csharp
// Use a custom log ID for the logger.
var factory = new MqttFactory();
var mqttClient = factory.CreateMqttClient(new MqttNetLogger("MyCustomId"));
```

# Advanced trace
In cases where the trace must be forwarded to other trace libraries like _SeriLog_ or _Microsoft.Extensions.Logging_ etc. a custom logger must be implemented and also implementing the interface _IMqttNetLogger_. Then the new logger can be specified like in chapter "Extended Trace".