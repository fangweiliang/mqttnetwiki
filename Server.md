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
var optionsBuilder = new MqttServerOptionsBuilder()
    .WithConnectionValidator(c =>
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
});
```

# Using a certificate
In order to use an encrypted connection a certificate __including__ the private key is required. The following code shows how to start a server using a certificate for encryption:
```csharp
using System.Reflection;
using System.Security.Authentication;
using System.Security.Cryptography.X509Certificates;
...

var currentPath = Path.GetDirectoryName(Assembly.GetExecutingAssembly().Location);
var certificate = new X509Certificate2(Path.Combine(currentPath, "certificate.pfx"),"yourPassword", X509KeyStorageFlags.Exportable);

var optionsBuilder = new MqttServerOptionsBuilder()
    .WithEncryptedEndpoint()
    .WithEncryptedEndpointPort(config.Port)
    .WithEncryptionCertificate(certificate.Export(X509ContentType.Pfx))
    .WithEncryptionSslProtocol(SslProtocols.Tls12)
```
But also other overloads getting a valid certificate blob (byte array) can be used.

For creating a self-signed certificate for testing the following command can be used (Windows SDK must be installed):

`makecert.exe -sky exchange -r -n "CN=selfsigned.crt" -pe -a sha1 -len 2048 -ss My "test.cer"`

OpenSSL can also be used to create a self-signed PFX certificate [as described here](https://github.com/Azure/azure-xplat-cli/wiki/Getting-Self-Signed-SSL-Certificates-(.pem-and-.pfx)).

## Example:
```
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365
openssl pkcs12 -export -out certificate.pfx -inkey key.pem -in cert.pem
```

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
var optionsBuilder = new MqttServerOptionsBuilder()
    .WithApplicationMessageInterceptor(context =>
    {
        if (context.ApplicationMessage.Topic == "my/custom/topic")
        {
            context.ApplicationMessage.Payload = Encoding.UTF8.GetBytes("The server injected payload.");
        }

        // It is possible to disallow the sending of messages for a certain client id like this:
        if (context.ClientId != "Someone")
        {
            c.AcceptPublish = false;
            return;
        }
        // It is also possible to read the payload and extend it. For example by adding a timestamp in a JSON document.
        // This is useful when the IoT device has no own clock and the creation time of the message might be important.
    })
    .Build();
```

If you want to stop processing an application message completely (like a delete) then the property _context.ApplicationMessage.Payload_ must be set to _null_.

# Intercepting subscriptions
A custom interceptor can be set to control which topics can be subscribed by a MQTT client. This allows moving private API-Topics to a protected area which is only available for certain clients. The following code shows how to use the subscription interceptor.
```csharp
// Protect several topics from being subscribed from every client.
var optionsBuilder = new MqttServerOptionsBuilder()
    .WithSubscriptionInterceptor(context =>
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
    })
    .Build();
```

It is also supported to use an async method instead of a synchronized one like in the above example.

# Storing data in the session
From version 3.0.6 and up, there is a `Dictionary<object, object>` called `SessionItems`. It allows to store custom data in the session and is available in all interceptors:

```csharp
var optionsBuilder = new MqttServerOptionsBuilder()
.WithConnectionValidator(c => { c.SessionItems.Add("SomeData", true); }
.WithSubscriptionInterceptor(c => { c.SessionItems.Add("YourData", new List<string>{"a", "b"}); }
.WithApplicationMessageInterceptor(c => { c.SessionItems.Add("Test", 123); }
```

# ASP.NET Core Integration

## ASP.NET Core 2.0

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
## ASP.NET Core 2.1+

_MQTTnet.AspNetCore_ is compatible with the abstractions present in ASP.NET Core 2.0 but it also offers a new tcp transport based on ASP.NET Core 2.1 Microsoft.AspNetCore.Connections.Abstractions. This transport is mutual exclusive with the old tcp transport so you may only add and use one of them. Our benchmark indicates that the new transport is up to 30 times faster. 

```csharp
// In class _Program_ of the ASP.NET Core 2.1 or 2.2 project.
private static IWebHost BuildWebHost(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseKestrel(o => {
            o.ListenAnyIP(1883, l => l.UseMqtt()); // mqtt pipeline
            o.ListenAnyIP(5000); // default http pipeline
        })
    .UseStartup<Startup>()
    .Build();

// In class _Startup_ of the ASP.NET Core 2.1 or 2.2 project.
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

# Windows IoT Core and UWP localhost loopback addresses
In Windows IoT Core as well as in UWP, loopback connections (127.0.0.1) are not allowed. If you try to connect to a locally running server (broker), this will fail.