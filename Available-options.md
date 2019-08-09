# Client
## Client options

|Method name|Description|Default value|
|-|-|-|
|`Build()`|Builds the options class.|Does not apply here.|
|`WithAuthentication(string method, byte[] data)`|Allows to use different authentication modes.|`null`|
|`WithCleanSession(bool value = true)`|Allows to use the client with MQTT clean session support.|`true`|
|`WithClientId(string value)`|Sets the used client id.|`Guid.NewGuid().ToString("N");`|
|`WithCommunicationTimeout(TimeSpan value)`|Sets the communication timeout.|`TimeSpan.FromSeconds(10.0)`|
|`WithCredentials(string username, string password)`|Sets the login credentials.|`null`, required.|
|`WithCredentials(string username, byte[] password)`|Sets the login credentials.|`null`, required.|
|`WithExtendedAuthenticationExchangeHandler(IMqttExtendedAuthenticationExchangeHandler handler)`|Allows to handle authentication in a custom way.|`null`|
|`WithKeepAlivePeriod(TimeSpan value)`|Sets the keep-alive period.|`TimeSpan.FromSeconds(15.0)`|
|`WithKeepAliveSendInterval(TimeSpan value)`|Sets the keep-alive send interval.|`null`|
|`WithMaximumPacketSize(uint? maximumPacketSize)`|Sets the maximum packet size.|`null`|
|`WithNoKeepAlive()`|Tells the client to use no keep-alive.|Does not apply here.|
|`WithProtocolVersion(MqttProtocolVersion value)`|Sets the MQTT protocol version.|`MqttProtocolVersion.V311`|
|`WithProxy(Action<MqttClientWebSocketProxyOptions> optionsBuilder)`|Sets the proxy.|`null`|
|`WithProxy(string address, string username = null, string password = null, string domain = null, bool bypassOnLocal = false, string[] bypassList = null)`|Sets the proxy.|`null`|
|`WithTls()`|Tells the client to use SSL / TLS.|Does not apply here.|
|`WithTls(MqttClientOptionsBuilderTlsParameters parameters)`|Tells the client to use SSL / TLS.|`null`|
|`WithTls(Action<MqttClientOptionsBuilderTlsParameters> optionsBuilder)`|Tells the client to use SSL / TLS.|`null`|
|`WithTopicAliasMaximum(ushort? topicAliasMaximum)`|Tells the client to allow a maximum number of topic aliases.|`null`|
|`WithReceiveMaximum(ushort? receiveMaximum)`|Tells the client to allow a maximum number of received packets.|`null`|
|`WithRequestProblemInformation(bool? requestProblemInformation = true)`|Tells the client to show request problem information.|`true`|
|`WithRequestResponseInformation(bool? requestResponseInformation = true)`|Tells the client to show request response problem information.|`true`|
|`WithSessionExpiryInterval(uint? sessionExpiryInterval)`|Tells the client to expire sessions after a amount of time.|`null`|
|`WithTcpServer(string server, int? port = null)`|Tells the client which MQTT broker to connect to (over TCP).|`null`, required. If no port is set, the default port `1883` is used (or `8883` for SSL).|
|`WithTcpServer(Action<MqttClientTcpOptions> optionsBuilder)`|Tells the client which MQTT broker to connect to (over TCP).|`null`, required. If no port is set, the default port `1883` is used (or `8883` for SSL).|
|`WithWebSocketServer(string uri, MqttClientOptionsBuilderWebSocketParameters parameters = null)`|Tells the client which MQTT broker to connect to (over WebSockets).|`null`, required.|
|`WithWebSocketServer(Action<MqttClientWebSocketOptions> optionsBuilder)`|Tells the client which MQTT broker to connect to (over WebSockets).|`null`, required.|
|`WithWillMessage(MqttApplicationMessage value)`|Tells the client which last will message will be sent.|`null`|
|`WithWillDelayInterval(uint? willDelayInterval)`|Tells the client which last will delay interval will be used.|`null`|

* Remarks: Either `WithTcpServer` or `WithWebSocketServer` or both are required, of course.

## Client methods and properties

|Method name|Description|Default value|
|`bool IsConnected`|A value indicating whether the client is connected or not.|`false`|
|`IMqttClientOptions Options`|The client options set to the client.|`null`|
|`IMqttClientConnectedHandler ConnectedHandler`|The connected handler to perform actions when the connection is established.|`null`|
|`IMqttClientDisconnectedHandler DisconnectedHandler`|The disconnected handler to perform actions when the connection is lost.|`null`|
|`Task<MqttClientAuthenticateResult> ConnectAsync(IMqttClientOptions options, CancellationToken cancellationToken)`|A method to connect the client.|Does not apply here.|
|`Task SendExtendedAuthenticationExchangeDataAsync(MqttExtendedAuthenticationExchangeData data, CancellationToken cancellationToken`|Sends extended authentication data.|Does not apply here.|
|`Task<MqttClientSubscribeResult> SubscribeAsync(MqttClientSubscribeOptions options, CancellationToken cancellationToken)`|Subscribes the client to a topic.|Does not apply here.|
|`Task<MqttClientUnsubscribeResult> UnsubscribeAsync(MqttClientUnsubscribeOptions options, CancellationToken cancellationToken)`|Unsubscribes the client from a topic.|Does not apply here.|

# ManagedClient
## ManagedClient options

|Method name|Description|-|
|-|-|-|
|`Build()`|Builds the options class.|Does not apply here.|
|`WithAutoReconnectDelay(TimeSpan value)`|Sets the auto-reconnect delay.|`TimeSpan.FromSeconds(5.0)`|
|`WithClientOptions(IMqttClientOptions value)`|Tells the managed client to use the configuration available under [Client options](https://github.com/chkr1011/MQTTnet/wiki/Available-options#client-options)|`null`|
|`WithClientOptions(MqttClientOptionsBuilder builder)`|Tells the managed client to use the configuration available under [Client options](https://github.com/chkr1011/MQTTnet/wiki/Available-options#client-options)|`null`|
|`WithClientOptions(Action<MqttClientOptionsBuilder> options)`|Tells the managed client to use the configuration available under [Client options](https://github.com/chkr1011/MQTTnet/wiki/Available-options#client-options)|`null`|
|`WithMaxPendingMessages(int value)`|Tells the client to use the number as maximum amount of pending messages.|`int.MaxValue`|
|`WithPendingMessagesOverflowStrategy(MqttPendingMessagesOverflowStrategy value)`|Tells the client which message overflow strategy should be used.|`MqttPendingMessagesOverflowStrategy.DropNewMessage`|
|`WithStorage(IManagedMqttClientStorage value)`|Tells the client which storage to use.|`null`|

# Server

|Method name|Description|
|-|-|
|-|-|