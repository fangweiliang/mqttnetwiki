# Simple connection
The following codes shows how to connect with a server:

```csharp
var options = new MqttClientOptions
{
    Server = "localhost"
};

var client = new MqttClientFactory().CreateMqttClient(options);
client.ApplicationMessageReceived += (s, e) =>
{
    Console.WriteLine("### RECEIVED APPLICATION MESSAGE ###");
    Console.WriteLine($"+ Topic = {e.ApplicationMessage.Topic}");
    Console.WriteLine($"+ Payload = {Encoding.UTF8.GetString(e.ApplicationMessage.Payload)}");
    Console.WriteLine($"+ QoS = {e.ApplicationMessage.QualityOfServiceLevel}");
    Console.WriteLine($"+ Retain = {e.ApplicationMessage.Retain}");
    Console.WriteLine();
};

client.Connected += async (s, e) =>
{
    Console.WriteLine("### CONNECTED WITH SERVER, SUBSCRIBING ###");

    await client.SubscribeAsync(new List<TopicFilter>
    {
        new TopicFilter("#", MqttQualityOfServiceLevel.AtMostOnce)
    });
};

client.Disconnected += async (s, e) => 
{
    Console.WriteLine("### DISCONNECTED FROM SERVER ###");
    await Task.Delay(TimeSpan.FromSeconds(5));

    try
    {
        await client.ConnectAsync();
    }
    catch
    {
        Console.WriteLine("### RECONNECTING FAILED ###");
    }
};

try
{
    await client.ConnectAsync();
}
catch
{
    Console.WriteLine("### CONNECTING FAILED ###");
}

Console.WriteLine("### WAITING FOR APPLICATION MESSAGES ###");

var messageFactory = new MqttApplicationMessageFactory();
while (true)
{
    Console.ReadLine();

    var applicationMessage = messageFactory.CreateApplicationMessage("myTopic", "Hello World", MqttQualityOfServiceLevel.AtLeastOnce);
    await client.PublishAsync(applicationMessage);
}
```

