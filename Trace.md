Debug messages for the _MqttClient_ and _MqttServer_ can be attached using a static event. The following code shows how to write all trace messages to the console window:

```csharp
MqttNetTrace.TraceMessagePublished += (s, e) =>
{
    Console.WriteLine($">> [{e.ThreadId}] [{e.Source}] [{e.Level}]: {e.Message}");
    if (e.Exception != null)
    {
        Console.WriteLine(e.Exception);
    }
};
```