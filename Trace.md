# Simple trace
Debug messages for the _MqttClient_ and _MqttServer_ can be attached using a static event. The following code shows how to write all trace messages to the console window:

```csharp
MqttNetTrace.TraceMessagePublished += (s, e) =>
{
    Console.WriteLine($">> [{e.TraceMessage.Timestamp:O}] [{e.TraceMessage.ThreadId}] [{e.TraceMessage.Source}] [{e.TraceMessage.Level}]: {e.TraceMessage.Message}");
    if (e.TraceMessage.Exception != null)
    {
        Console.WriteLine(e.TraceMessage.Exception);
    }
};
```
# Advanced trace
For use cases where multiple clients are running and trace messages must be separated from each other it is possible to inject a custom trace handler. This must be done at factory level of the client or server. The following code shows a custom trace handler:
```csharp
public class CustomTraceHandler : IMqttNetTraceHandler
{
    private readonly string _clientId;

    public CustomTraceHandler(string clientId)
    {
        _clientId = clientId;
    }

    public bool IsEnabled { get; } = true;

    public void HandleTraceMessage(MqttNetTraceMessage traceMessage)
    {
        // Client ID is added to the trace message.
        Console.WriteLine($">> [{_clientId}] [{traceMessage.Timestamp:O}] [{traceMessage.ThreadId}] [{traceMessage.Source}] [{traceMessage.Level}]: {traceMessage.Message}");
        if (traceMessage.Exception != null)
        {
            Console.WriteLine(traceMessage.Exception);
        }
    }
}
```
This above trace handler can be injected into a client or server using the following code:
```csharp
var client = new MqttClientFactory().CreateMqttClient(new CustomTraceHandler("Client 1"));
```
