# Preparation
Creating a MQTT server is similar to creating a MQTT client. The following code shows the most simple way of creating a new MQTT server with a TCP endpoint which is listening at the default port 1883.
```csharp
// Start a MQTT server.
var mqttServer = new MqttFactory().CreateMqttServer();
await mqttServer.StartAsync(new MqttServerOptions());
Console.WriteLine("Press any key to exit.");
Console.ReadLine();
await mqttServer.StopAsync();
```

Setting several options for the MQTT server is possible by setting the property values of the _MqttServerOptions_ directly or via using the _MqttServerOptionsBuilder_ (which is recommended). The following code shows how to use the _MqttServerOptionsBuilder_.
```csharp
// Configure MQTT server.
// Configure MQTT server.
var optionsBuilder = new MqttServerOptionsBuilder()
    .WithConnectionBacklog(100)
    .WithDefaultEndpointPort(1884);

var mqttServer = new MqttFactory().CreateMqttServer();
await mqttServer.StartAsync(optionsBuilder.Build());
```

# Validating MQTT clients
The following code shows how to validate an incoming MQTT client connection request:
```csharp
// Setup client validator.
var options = new MqttServerOptions();

options.ConnectionValidator = c =>
{
    if (c.ClientId.Length < 10)
    {
        c.ReturnCode = MqttConnectReturnCode.ConnectionRefusedIdentifierRejected;
        return;
    }

    if (c.Username != "mySecretUser")
    {
        c.ReturnCode = MqttConnectReturnCode.ConnectionRefusedBadUsernameOrPassword;
        return;
    }

    if (c.Password != "mySecretPassword")
    {
        c.ReturnCode = MqttConnectReturnCode.ConnectionRefusedBadUsernameOrPassword;
        return;
    }

    c.ReturnCode = MqttConnectReturnCode.ConnectionAccepted;
};
```

# Using a certificate
In order to use an encrypted connection a certificate __including__ the private key is required. The following code shows how to start a server using a certificate for encryption:
```csharp
using System.Security.Cryptography.X509Certificates
...
var certificate = new X509Certificate2(@"C:\certs\test\test.cer", password, X509KeyStorageFlags.Exportable);
options.TlsEndpointOptions.Certificate = certificate.Export(X509ContentType.Pfx);
options.TlsEndpointOptions.IsEnabled = true;
```
But also other overloads getting a valid certificate blob (byte array) can be used.

For creating a self-signed certificate for testing the following command can be used (Windows SDK must be installed):

`makecert.exe -sky exchange -r -n "CN=selfsigned.crt" -pe -a sha1 -len 2048 -ss My "test.cer"`

OpenSSL can can also be used to create a self-signed PFX certificate [as described here](https://github.com/Azure/azure-xplat-cli/wiki/Getting-Self-Signed-SSL-Certificates-(.pem-and-.pfx)). 

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

If you want to stop processing an application message completely (like a delete) then the property _context.ApplicationMessage.Payload_ must be set to _null_.

# Intercepting subscriptions
A custom interceptor can be set to control which topics can be subscribed by a MQTT client. This allows moving private API-Topics to a protected area which is only available for certain clients. The following code shows how to use the subscription interceptor.
```csharp
// Protect several topics from being subscribed from every client.
options.SubscriptionInterceptor = context =>
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

# ASP.NET Core Integration
This library also has support for a WebSocket based server which is integrated into ASP.NET Core 2.0. This functionality requires an additional library called _MQTTnet.AspNetCore_. After adding this library a MQTT server can be added to a Kestrel HTTP server. 

```csharp
// In class _Startup_ of the ASP.NET Core 2.0 project.
public void ConfigureServices(IServiceCollection services)
{
     //this adds a hosted mqtt server to the services
     services.AddHostedMqttServer(builder => builder.WithDefaultEndpointPort(1883));

     //this adds tcp server support based on System.Net.Socket
     services.AddMqttTcpServerAdapter();

     //this adds websocket support
     services.AddMqttWebSocketServerAdapter();
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    // this maps the websocket to an mqtt endpoint
    app.UseMqttEndpoint();
    // other stuff
}
```
## ASP.NET Core 2.1

_MQTTnet.AspNetCore_ is compatible with the abstractions present in ASP.NET Core 2.0 but it also offers a new tcp transport based on  ASP.NET Core 2.1 Microsoft.AspNetCore.Connections.Abstractions. This transport is mutual exclusive with the old tcp transport so you may only add and use one of them. Our benchmark indicates that the new transport is up to 30 times faster. 

```csharp
// In class _Program_ of the ASP.NET Core 2.1 project.
private static IWebHost BuildWebHost(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseKestrel(o => {
            o.ListenAnyIP(1883, l => l.UseMqtt()); // mqtt pipeline
            o.ListenAnyIP(5000); // default http pipeline
        })
    .UseStartup<Startup>()
    .Build();

// In class _Startup_ of the ASP.NET Core 2.1 project.
public void ConfigureServices(IServiceCollection services)
{
     //this adds a hosted mqtt server to the services
     services.AddHostedMqttServer();

     //this adds tcp server support based on Microsoft.AspNetCore.Connections.Abstractions
     services.AddMqttConnectionHandler();

     //this adds websocket support
     services.AddMqttWebSocketServerAdapter();
}
```


