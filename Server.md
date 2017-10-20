# Preparation
The following code is required to run a simple MQTT server (broker) with default options (Port etc.):
```csharp
var options = new MqttServerOptions();

var mqttServer = new MqttServerFactory().CreateMqttServer(options);
await mqttServer.StartAsync();

Console.WriteLine("Press any key to exit.");
Console.ReadLine();

await mqttServer.StopAsync();
```

# Validating MQTT clients
The following code shows how to validate an incoming MQTT client connection request:
```csharp
var options = new MqttServerOptions
{
    ConnectionValidator = c =>
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
    }
};
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
The server is also able to publish MQTT application messages. The object is the same as for the client implementation. Due to the fact that the server is able to publish its own messages it is not required having a loopback connection at the same process. Also this allows integrating the server into a Windows IoT Core UWP app. This platform has a network isolation which makes it impossible to communicate via localhost etc. Examples for publishing a message are described at the client section of this Wiki.

# Saving retained messages
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