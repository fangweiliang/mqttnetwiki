# Preparation
The following code is required to setup a MQTT client:
```csharp
var mqttClient = new MqttClientFactory().CreateMqttClient();
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