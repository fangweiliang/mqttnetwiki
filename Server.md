# Preparation
Creating a MQTT server is similar to creating a MQTT client. The following code shows the most simple way of creating a new MQTT server with a TCP endpoint which is listening at the default port 1883.
```csharp
// Start a MQTT server.
var mqttServer = new MqttFactory().CreateMqttServer();
await mqttServer.StartAsync();
Console.WriteLine("Press any key to exit.");
Console.ReadLine();
await mqttServer.StopAsync();
```

Setting several options for the MQTT server is possible by providing a configuration callback. This callback exposes all available options which can be set within the callback. The following code shows how to setup a MQTT server.
```csharp
// Configure MQTT server.
var mqttServer = new MqttFactory().CreateMqttServer(options =>
{
    options.ConnectionBacklog = 100;
    options.DefaultEndpointOptions.Port = 1884;
    options.ConnectionValidator = packet =>
    {
        if (packet.ClientId != "Highlander")
        {
            return MqttConnectReturnCode.ConnectionRefusedIdentifierRejected;
        }

        return MqttConnectReturnCode.ConnectionAccepted;
    };
});
```

It is also possible to use the service collection of the DI directly. The following code shows how to use the DI components directly.
```csharp
//setup
var services = new ServiceCollection()
    .AddLogging()
    .AddMqttServer(options => 
    {
       // modify options here
    })
    .BuildServiceProvider();

//use 
var mqttServer = services.GetRequiredService<IMqttServer>();
await mqttServer.StartAsync();
Console.WriteLine("Press any key to exit.");
Console.ReadLine();
await mqttServer.StopAsync();
```

# Validating MQTT clients
The following code shows how to validate an incoming MQTT client connection request:
```csharp
// Setup client validator.
var mqttServer = new MqttFactory().CreateMqttServer(options =>
{
    options.ConnectionValidator = c =>
    {
        if (c.ClientId.Length < 10)
        {
            return MqttConnectReturnCode.ConnectionRefusedIdentifierRejected;
        }

        if (c.Username != "mySecretUser")
        {
            return MqttConnectReturnCode.ConnectionRefusedBadUsernameOrPassword;
        }

        if (c.Password != "mySecretPassword")
        {
            return MqttConnectReturnCode.ConnectionRefusedBadUsernameOrPassword;
        }

        return MqttConnectReturnCode.ConnectionAccepted;
    };
});
```

# Using a certificate
In order to use an encrypted connection a certificate __including__ the private key is required. The following code shows how to start a server using a certificate for encryption:
```csharp
var certificate = new X509Certificate(@"C:\certs\test\test.cer", "");
options.TlsEndpointOptions.Certificate = certificate.Export(X509ContentType.Cert);
```
But also other overloads getting a valid certificate blob (byte array) can be used.

For creating a self signed certificate for testing the following command can be used (Windows SDK must be installed):

`makecert.exe -sky exchange -r -n "CN=selfsigned.crt" -pe -a sha1 -len 2048 -ss My "test.cer"`

# Publishing messages
The server is also able to publish MQTT application messages. The object is the same as for the client implementation. Due to the fact that the server is able to publish its own messages it is not required having a loopback connection in the same process. 

This allows also running the server in a Windows IoT Core UWP app. This platform has a network isolation which makes it impossible to communicate via localhost etc. 

Examples for publishing a message are described at the client section of this Wiki.

# Consuming messages
The server is also able to process every application message which was published by any client. The event _ApplicationMessageReceived_ will be fired for every processed message. It has the same format as for the client but additionally has the _ClientId_. 

Details for consuming a application messages are described at the client section of this Wiki.

# Saving retained application messages
The server supports retained MQTT messages. Those messages are kept and send to clients when they connect and subscribe to them. It is also supported to save all retained messages and loading them after the server has started. This required implementing an interface. The following code shows how to serialize retained messages as JSON:
```csharp
// Setting the options
options.Storage = new RetainedMessageHandler();

// The implementation of the storage:
// This code uses the JSON library "Newtonsoft.Json".
public class RetainedMessageHandler : IMqttServerStorage
{
    private const string Filename = "C:\\MQTT\\RetainedMessages.json";

    public Task SaveRetainedMessagesAsync(IList<MqttApplicationMessage> messages)
    {
        File.WriteAllText(Filename, JsonConvert.SerializeObject(messages));
        return Task.FromResult(0);
    }

    public Task<IList<MqttApplicationMessage>> LoadRetainedMessagesAsync()
    {
        IList<MqttApplicationMessage> retainedMessages;
        if (File.Exists(Filename))
        {
            var json = File.ReadAllText(Filename);
            retainedMessages = JsonConvert.DeserializeObject<List<MqttApplicationMessage>>(json);
        }
        else
        {
            retainedMessages = new List<MqttApplicationMessage>();
        }
            
        return Task.FromResult(retainedMessages);
    }
}
```
# Intercepting application messages
A custom interceptor can be set at the server options. This interceptor is called for __every__ application message which is received by the server. This allows extending application messages __before__ they are persisted (in case of a retained message) __and before__ being dispatched to subscribers. This allows use cases like adding a time stamp to every application message if the hardware device does not know the time or time zone etc. The following code shows how to use the interceptor:
```csharp
// Extend the timestamp for all messages from clients.
options.ApplicationMessageInterceptor = context =>
{
    if (MqttTopicFilterComparer.IsMatch(context.ApplicationMessage.Topic, "/myTopic/WithTimestamp/#"))
    {
        // Replace the payload with the timestamp. But also extending a JSON 
        // based payload with the timestamp is a suitable use case.
        context.ApplicationMessage.Payload = Encoding.UTF8.GetBytes(DateTime.Now.ToString("O"));
    }
};
```

# Intercepting subscriptions
A custom interceptor can be set to control which topics can be subscribed by a MQTT client. This allows moving private API-Topics to a protected are which is only available for certain clients. The following code shows how to use the subscription interceptor.
```csharp
// Protect several topics from being subscribed from every client.
options.SubscriptionsInterceptor = context =>
{
    if (context.TopicFilter.Topic.StartsWith("admin/foo/bar") && context.ClientId != "theAdmin")
    {
        context.AcceptSubscription = false;
    }

    if (context.TopicFilter.Topic.StartsWith("the/secret/stuff") && context.ClientId != "Imperator")
    {
        context.AcceptSubscription = false;
        context.CloseConnection = true;
    }
};
```

# WebSocket endpoint
This library also has support for a WebSocket based server which is integrated into ASP.NET Core 2.0. This functionality requires an additional library called _MQTTnet.AspNetCore_. After adding this library a MQTT server can be added to a Kestrel HTTP server. The following code shows how to configure the WebSocket endpoint.
```csharp
// In class _Startup_ of the ASP.NET Core 2.0 project.
public void ConfigureServices(IServiceCollection services)
{
    services.AddHostedMqttServer();
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseMqttEndpoint();
    // other stuff
}
```