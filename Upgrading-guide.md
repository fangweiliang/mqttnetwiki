# From 2.8.5 to 3.0.0:

## General:
* `MqttProtocolVersion` is now in the `MQTTnet.Formatter` namespace.
* `MqttFixedHeader`, `MqttPacketBodyReader`, `MqttPacketReader`, `MqttPacketWriter` and `MqttProtocolVersion` are now in the `MQTTnet.Formatter` namespace.
* `IMqttPacketSerializer` and `MqttPacketSerializer` do not exist anymore. --> Wait for answer in https://github.com/chkr1011/MQTTnet/issues/614.

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
* Creation of the managed client changed from `var client = new MqttFactory().CreateManagedMqttClient();` to `?` --> Wait for answer in https://github.com/chkr1011/MQTTnet/issues/614.

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
