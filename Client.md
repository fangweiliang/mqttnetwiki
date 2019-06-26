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

## Securing passwords
The default implementation uses a `string` to hold the client password. However this is a security vulnerability because the password is stored in the heap as clear text. It is recommended to use a `SecureString` for this purpose. But this class is not available for all supported platforms (UWP, netstandard 1.3). This library does not implement it because for other platforms custom implementations like async encryption are required. It is recommended to implement an own _IMqttClientCredentials_ class which returns the decrypted password but does not store it unencrypted.

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
// For .NET Framework & netstandard apps:
var options = new MqttClientOptionsBuilder()
    .WithTls(new MqttClientOptionsBuilderTlsParameters
    {
        CertificateValidationCallback = (X509Certificate x, X509Chain y, SslPolicyErrors z, IMqttClientOptions o) =>
            {
                // TODO: Check conditions of certificate by using above parameters.
                return true;
            }
    })
    .Build();

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

Also secure web socket connections can be used via calling the _UseTls()_ method which will switch the protocol from _ws://_ to _wss://_. Usually the sub protocol is required which can be added to the URI directly or to a dedicated property.

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
client.UseDisconnectedHandler(async e =>
{
    Console.WriteLine("### DISCONNECTED FROM SERVER ###");
    await Task.Delay(TimeSpan.FromSeconds(5));

    try
    {
        await client.ConnectAsync(options);
    }
    catch
    {
        Console.WriteLine("### RECONNECTING FAILED ###");
    }
});
```
# Consuming messages
The following code shows how to handle incoming messages:
```csharp
client.UseApplicationMessageReceivedHandler(e =>
{
    Console.WriteLine("### RECEIVED APPLICATION MESSAGE ###");
    Console.WriteLine($"+ Topic = {e.ApplicationMessage.Topic}");
    Console.WriteLine($"+ Payload = {Encoding.UTF8.GetString(e.ApplicationMessage.Payload)}");
    Console.WriteLine($"+ QoS = {e.ApplicationMessage.QualityOfServiceLevel}");
    Console.WriteLine($"+ Retain = {e.ApplicationMessage.Retain}");
    Console.WriteLine();

    Task.Run(() => client.PublishAsync("hello/world"));
});
```
It is also supported to use an async method instead of a synchronized one like in the above example.

⚠️ **Publishing messages inside that received messages handler requires to use _Task.Run_ when using a QoS > 0. The reason is that the message handler has to finish first before the next message is received. The reason is to preserve ordering of the application messages.** 

# Subscribing to a topic
Once a connection with the server is established subscribing to a topic is possible. The following code shows how to subscribe to a topic after the MQTT client has connected.
~~~csharp
client.UseConnectedHandler(async e =>
{
    Console.WriteLine("### CONNECTED WITH SERVER ###");

    // Subscribe to a topic
    await client.SubscribeAsync(new TopicFilterBuilder().WithTopic("my/topic").Build());

    Console.WriteLine("### SUBSCRIBED ###");
});
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

# RPC calls
The extension _MQTTnet.Extensions.Rpc_ (available as nuget) allows sending a request and waiting for the matching reply. This is done via defining a pattern which uses the topic to correlate the request and the response. From client usage it is possible to define a timeout. The following code shows how to send a RPC call.

```csharp
var rpcClient = new MqttRpcClient(_mqttClient);

var timeout = TimeSpan.FromSeconds(5);
var qos = MqttQualityOfServiceLevel.AtMostOnce;

var response = await rpcClient.ExecuteAsync(timeout, "myMethod", payload, qos);
```

The device (Arduino, ESP8266 etc.) which responds to the request needs to parse the topic and reply to it. The following code shows how to implement the handler.

```C
// If using the MQTT client PubSubClient it must be ensured 
// that the request topic for each method is subscribed like the following.
_mqttClient.subscribe("MQTTnet.RPC/+/ping");
_mqttClient.subscribe("MQTTnet.RPC/+/do_something");

// It is not allowed to change the structure of the topic.
// Otherwise RPC will not work.
// So method names can be separated using an _ or . but no +, # or /.
// If it is required to distinguish between devices
// own rules can be defined like the following:
_mqttClient.subscribe("MQTTnet.RPC/+/deviceA.ping");
_mqttClient.subscribe("MQTTnet.RPC/+/deviceB.ping");
_mqttClient.subscribe("MQTTnet.RPC/+/deviceC.getTemperature");

// Within the callback of the MQTT client the topic must be checked
// if it belongs to MQTTnet RPC. The following code shows one
// possible way of doing this.
void mqtt_Callback(char *topic, byte *payload, unsigned int payloadLength)
{
	String topicString = String(topic);

	if (topicString.startsWith("MQTTnet.RPC/")) {
		String responseTopic = topicString + String("/response");

		if (topicString.endsWith("/deviceA.ping")) {
			mqtt_publish(responseTopic, "pong", false);
			return;
		}
	}
}

// Important notes:
// ! Do not send response message with the _retain_ flag set to true.
// ! All required data for a RPC call and the result must be placed into the payload.
```

# Connecting with Amazon AWS
Due to a known issue with WebSocket implementation for .NET it is not possible to connect with Amazon AWS with MQTTnet because it relies on the .NET implementation of WebSockets.

But there is a sample available which shows how to use a different transport layer implementation.

https://github.com/aws-samples/aws-iot-core-dotnet-app-mqtt-over-websockets-sigv4