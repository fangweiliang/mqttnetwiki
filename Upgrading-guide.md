# From 3.0.5 to 3.0.6:

## General:
* There is a new `Dictionary<object, object>` called `SessionItems`. It allows to store custom data in the session and is available in all interceptors. For more examples, check https://github.com/chkr1011/MQTTnet/wiki/Server#storing-data-in-the-session.

# From 3.0.3 to 3.0.5:

## General:
* `MqttConnectReturnCode` is now deprecated, use `MqttConnectReasonCode` instead.

# From 2.8.5 to 3.0.0:

## General:
* `MqttProtocolVersion` is now in the `MQTTnet.Formatter` namespace.
* `MqttFixedHeader`, `MqttPacketBodyReader`, `MqttPacketReader`, `MqttPacketWriter` and `MqttProtocolVersion` are now in the `MQTTnet.Formatter` namespace.
* `IMqttPacketSerializer` and `MqttPacketSerializer` do not exist anymore.

## ManagedClient:
* Updated async handlers:
```csharp
private void Something()
{
    mqttClient.ApplicationMessageReceivedHandler = new MqttApplicationMessageReceivedHandlerDelegate(OnAppMessage);
    mqttClient.ConnectedHandler = new MqttClientConnectedHandlerDelegate(OnConnected);
    mqttClient.DisconnectedHandler = new MqttClientDisconnectedHandlerDelegate(OnDisconnected);
    mqttClient.ConnectingFailedHandler = new ConnectingFailedHandlerDelegate(OnConnectingFailed);
}

private async void OnAppMessage(MqttApplicationMessageReceivedEventArgs e)
{
}

private async void OnConnected(MqttClientConnectedEventArgs e)
{
}

private async void OnDisconnected(MqttClientDisconnectedEventArgs e)
{
}

private async void OnConnectingFailed(ManagedProcessFailedEventArgs e)
{
}
```

## Client:
* Updated async handlers:
```csharp
private void Something()
{
    mqttClient.ApplicationMessageReceivedHandler = new MqttApplicationMessageReceivedHandlerDelegate(OnAppMessage);
    mqttClient.ConnectedHandler = new MqttClientConnectedHandlerDelegate(OnConnected);
    mqttClient.DisconnectedHandler = new MqttClientDisconnectedHandlerDelegate(OnDisconnected);
}

private async void OnAppMessage(MqttApplicationMessageReceivedEventArgs e)
{
}

private async void OnConnected(MqttClientConnectedEventArgs e)
{
}

private async void OnDisconnected(MqttClientDisconnectedEventArgs e)
{
}
```
