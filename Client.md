# Preparation
The following code is required to setup a MQTT client:

## before version 2.5
```csharp
var mqttClient = new MqttClientFactory().CreateMqttClient();
```
## from Version 2.5 on

With Version 2.5 Microsoft.Extensions.DependencyInjection was introduced so you have two options for creating the client now. 

### Option a 

Use the new MqttFactory that knows all services Mqtt needs.

```csharp
var mqttClient = new MqttFactory().CreateMqttClient();
```

### Option b

Use a service collection and constructor injection. 

```csharp
//setup
var services = new ServiceCollection()
    .AddMqttClient()
    .BuildServiceProvider();

//use 
var mqttClient = services.GetRequiredService<IMqttClient>();

//or
public class ServiceClass
{
    ServiceClass(IMqttClient mqttclient) 
    {
    }
}

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