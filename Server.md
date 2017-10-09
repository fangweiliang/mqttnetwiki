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
