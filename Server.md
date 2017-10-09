# Simple server
The following code is required to start a simple MQTT server (broker):
```csharp
var options = new MqttServerOptions
{
    ConnectionValidator = p =>
    {
        if (p.ClientId == "SpecialClient")
        {
            if (p.Username != "USER" || p.Password != "PASS")
            {
                return MqttConnectReturnCode.ConnectionRefusedBadUsernameOrPassword;
            }
        }

        return MqttConnectReturnCode.ConnectionAccepted;
    }
};

var mqttServer = new MqttServerFactory().CreateMqttServer(options);
mqttServer.Start();

Console.WriteLine("Press any key to exit.");
Console.ReadLine();

mqttServer.Stop();
```