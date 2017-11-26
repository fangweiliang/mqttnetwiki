# Preparation
The following code shows how to create a new MQTT client in the most simple way using the _MqttFactory_.
```csharp
// Create a new MQTT client.
var factory = new MqttFactory();
var mqttClient = factory.CreateMqttClient();
```

# Client options
All options for the MQTT client are bundled in one class named _MqttClientOptions_. It is possible to fill options manually in code via the properties but it is recommended to use the _MqttClientOptionsBuilder_. This class provides a _fluent API_ and allows setting the options easily by providing several overloads and helper methods. The following code shows how to use the builder with several random options.
```csharp
// Create TCP based options using the builder.
var options = new MqttClientOptionsBuilder()
    .WithClientId("Client1")
    .WithTcpServer("broker.hivemq.com")
    .WithCredentials("bud", "%spencer%")
    .WithTls()
    .WithCleanSession()
    .Build();
```

# TCP connection
The following code shows how to set the options of the MQTT client to make use of a TCP based connection.
```csharp
// Use TCP connection.
var options = new MqttClientOptionsBuilder()
    .WithTcpServer("broker.hivemq.com", 1883) // Port is optional
    .Build();
```

# Secure TCP connection
The following code shows how to use a TLS secured TCP connection (properties are only set as reference):
```csharp
// Use secure TCP connection.
var options = new MqttClientOptionsBuilder()
    .WithTcpServer("broker.hivemq.com")
    .WithTls()
    .Build();
```

# Dealing with special certificates
In order to deal with special certificate errors a special validation callback is available (.NET Framework & netstandard). For UWP apps a property is available.
```csharp
// For .NET Framwork & netstandard apps:
MqttTcpChannel.CustomCertificateValidationCallback = (x509Certificate, x509Chain, sslPolicyErrors, mqttClientTcpOptions) =>
{
    if (mqttClientTcpOptions.Server == "server_with_revoked_cert")
    {
        return true;
    }

    return false;
};

// For UWP apps:
MqttTcpChannel.CustomIgnorableServerCertificateErrorsResolver = o =>
{
    if (o.Server == "server_with_revoked_cert")
    {
        return new []{ ChainValidationResult.Revoked };
    }

    return new ChainValidationResult[0];
};
```

# WebSocket connection
In order to use a WebSocket communication channel the following code is required.
```csharp
// Use WebSocket connection.
var options = new MqttClientOptionsBuilder()
    .WithWebSocketServer("broker.hivemq.com:8000/mqtt")
    .Build();
```

# Connecting
After setting up the MQTT client options a connection can be established. The following code shows how to connect with a server.
```csharp
// Use WebSocket connection.
var options = new MqttClientOptionsBuilder()
    .WithWebSocketServer("broker.hivemq.com:8000/mqtt")
    .Build();

await client.ConnectAsync(options);
```

# Reconnecting
If the connection to the server is lost the _Disconnected_ event is fired. The event is also fired if a call to _ConnectAsync_ has failed because the server is not reachable etc. This allows calling the _ConnectAsync_ method only one time and dealing with retries etc. via consuming the _Disconnected_ event. If the reconnect fails the _Disconnected_ event is fired again. The following code shows how to setup this behavior including a short delay.
```csharp
mqttClient.Disconnected += async (s, e) =>
{
    Console.WriteLine("### DISCONNECTED FROM SERVER ###");
    await Task.Delay(TimeSpan.FromSeconds(5));

    try
    {
        await mqttClient.ConnectAsync(options);
    }
    catch
    {
        Console.WriteLine("### RECONNECTING FAILED ###");
    }
};
```
# Consuming messages
The following code shows how to handle incoming messages:
```csharp
mqttClient.ApplicationMessageReceived += (s, e) =>
{
    Console.WriteLine("### RECEIVED APPLICATION MESSAGE ###");
    Console.WriteLine($"+ Topic = {e.ApplicationMessage.Topic}");
    Console.WriteLine($"+ Payload = {Encoding.UTF8.GetString(e.ApplicationMessage.Payload)}");
    Console.WriteLine($"+ QoS = {e.ApplicationMessage.QualityOfServiceLevel}");
    Console.WriteLine($"+ Retain = {e.ApplicationMessage.Retain}");
    Console.WriteLine();
};
```

# Subscribing to a topic
Once a connection with the server is established subscribing to a topic is possible. The following code shows how to subscribe to a topic after the MQTT client has connected.
~~~csharp
client.Connected += async (s, e) =>
{
    Console.WriteLine("### CONNECTED WITH SERVER ###");

    // Subscribe to a topic
    await mqttClient.SubscribeAsync(new TopicFilterBuilder().WithTopic("my/topic").Build());

    Console.WriteLine("### SUBSCRIBED ###");
};
~~~

# Publishing messages
Application messages can be created using the properties directly or via using the _MqttApplicationMessageBuilder_. This class has some useful overloads which allows dealing with different payload formats easily. The API of the builder is a _fluent API_. The following code shows how to compose an application message and publishing them:
```csharp
var message = new MqttApplicationMessageBuilder()
    .WithTopic("MyTopic")
    .WithPayload("Hello World")
    .WithExactlyOnceQoS()
    .WithRetainFlag()
    .Build();

await client.PublishAsync(message);
```
It is not required to fill all properties of an application message. The following code shows how to create a very basic application message:
```csharp
var message = new MqttApplicationMessageBuilder()
    .WithTopic("/MQTTnet/is/awesome")
    .Build();
```