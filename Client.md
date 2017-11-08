# Preparation
This library uses the library _Microsoft.Extensions.DependencyInjection_ for DI which allows creating a MQTT client in several ways

The following code shows how to create a new MQTT client in the most simple way using the _MqttFactory_.
```csharp
// Create a new MQTT client
var factory = new MqttFactory();
var client = factory.CreateMqttClient();
```

It is also possible to use the service provider from the DI library directly and get an instance of the MQTT client. The following code shows how to use the service provider approach.
```csharp
// Create a client from the service provider manually.
var serviceProvider = new ServiceCollection()
    .AddMqttClient()
    .BuildServiceProvider();

var mqttClient = serviceProvider.GetRequiredService<IMqttClient>();
```

At least it is possible to let the DI inject the MQTT client in other services. In order to use this the _IMqttClient_ interface must be used as a parameter for the constructor. The following code shows how to declare the MQTT client for DI.
```csharp
public class ServiceClass
{
    ServiceClass(IMqttClient mqttclient) 
    {
    }
}

```

# Client options
All options for the MQTT client are bundled in one class named _MqttClientOptions_. It is possible to fill options manually in code via the properties but it is recommended to use the _MqttClientOptionsBuilder_. This class provides a fluent API and allows setting the options easily by providing several overloads and helper methods. The following code shows how to use the builder with several random options.
```csharp
// Create TCP based options using the builder
var options = new MqttClientOptionsBuilder()
    .WithClientId("Client1")
    .WithTcpServer("broker.hivemq.com")
    .WithCredentials("bud", "%spencer%")
    .WithTls()
    .WithCleanSession()
    .Build();
```


# TCP connection
The following code shows the options for a regular TCP connection to a MQTT server (broker):
```csharp
var tcpOptions = new MqttClientTcpOptions
{
    Server = "broker.hivemq.org",
    ClientId = "TestClient"
};

await mqttClient.ConnectAsync(tcpOptions);
```

# Using a secure TCP connection
The following code shows how to use a TLS secured TCP connection (properties are only set as reference):
```csharp
var secureTcpOptions = new MqttClientTcpOptions
{
    Server = "broker.hivemq.org",
    ClientId = "TestClient",
    TlsOptions = new MqttClientTlsOptions
    {
        UseTls = true,
        IgnoreCertificateChainErrors = true,
        IgnoreCertificateRevocationErrors = true,
        AllowUntrustedCertificates = true
    }
};
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
In order to use a WebSocket communication channel the following code is required:
```csharp
var wsOptions = new MqttClientWebSocketOptions
{
    Uri = "broker.hivemq.com:8000/mqtt",
    ClientId = "TestClient"
};

await mqttClient.ConnectAsync(wsOptions);
```

# Reconnecting
If the connection to the server is lost the _Disconnected_ event is fired. It can be used to reconnect (after a short delay). If the reconnect fails the _Disconnected_ event is fired again. The following code shows how to setup this behavior:
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

# Subscribing to a topic
Here's an example of subscribing to catch all messages:
~~~csharp
client.Connected += async (s, e) =>
{
	Console.WriteLine("### CONNECTED WITH SERVER ###");

	await client.SubscribeAsync(new List<TopicFilter>
	{
		new TopicFilter("#", MqttQualityOfServiceLevel.AtMostOnce)
	});

	Console.WriteLine("### SUBSCRIBED ###");
};
~~~

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

# Publishing messages
Application messages can be created using the constructor directly or via using the _MqttApplicationMessageBuilder_. This class has some useful overloads which allows dealing with different payload formats easily. The API of the builder is a _fluent API_. The following code shows how to compose an application message and publishing them:
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