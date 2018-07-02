# Preparation
This project also contains a _managed_ MQTT client. The client has some additional functionality in comparison with the regular _MqttClient_. Those functionalities are reflecting the most common use cases and thus the _ManagedClient_ provides a out-of-the-box MQTT client with the following features.
1. The managed client is _started_ once and will maintain the connection automatically including reconnecting etc.
2. All MQTT application messages are added to an internal queue and processed once the server is available.
3. All MQTT application messages can be stored to support sending them after a restart of the application
4. All subscriptions are managed across server connections. There is no need to subscribe manually after the connection with the server is lost.

The managed Client is available as a separate nuget package (https://www.nuget.org/packages/MQTTnet.Extensions.ManagedClient/).

The following code shows how to setup and start a managed MQTT client.
```csharp
// Setup and start a managed MQTT client.
var options = new ManagedMqttClientOptionsBuilder()
    .WithAutoReconnectDelay(TimeSpan.FromSeconds(5))
    .WithClientOptions(new MqttClientOptionsBuilder()
        .WithClientId("Client1")
        .WithTcpServer("broker.hivemq.com")
        .WithTls().Build())
    .Build();

var mqttClient = new MqttFactory().CreateManagedMqttClient();
await mqttClient.SubscribeAsync(new TopicFilterBuilder().WithTopic("my/topic").Build());
await mqttClient.StartAsync(options);
```

# Message processing
Similar to the regular client the managed client also has a _Publish_ method. But for the managed client this method will not throw an exception on failure because the message is only added to an internal queue and processed later.

But the managed client has several events which allows dealing with send errors etc.