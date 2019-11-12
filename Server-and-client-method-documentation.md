# Client
## Client options

|Method name|Description|Default value|
|-|-|-|
|`Build()`|Builds the options class.|Does not apply here.|
|`WithAuthentication(string method, byte[] data)`|Allows to use different authentication modes.|`null`|
|`WithCleanSession(bool value = true)`|Allows to use the client with MQTT clean session support.|`true`|
|`WithClientId(string value)`|Sets the used client id.|`Guid.NewGuid().ToString("N")`|
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
|-|-|-|
|`IMqttClientConnectedHandler ConnectedHandler`|The connected handler to perform actions when the connection is established.|`null`|
|`IMqttClientDisconnectedHandler DisconnectedHandler`|The disconnected handler to perform actions when the connection is lost.|`null`|
|`bool IsConnected`|A value indicating whether the client is connected or not.|`false`|
|`IMqttClientOptions Options`|The client options set to the client.|`null`|
|`Task<MqttClientAuthenticateResult> ConnectAsync(IMqttClientOptions options, CancellationToken cancellationToken)`|A method to connect the broker.|Does not apply here.|
|`Task<MqttClientPublishResult> PublishAsync(MqttApplicationMessage applicationMessage, CancellationToken cancellationToken)`|Publishes a message to a topic.|Does not apply here.|
|`Task SendExtendedAuthenticationExchangeDataAsync(MqttExtendedAuthenticationExchangeData data, CancellationToken cancellationToken`|Sends extended authentication data.|Does not apply here.|
|`Task<MqttClientSubscribeResult> SubscribeAsync(MqttClientSubscribeOptions options, CancellationToken cancellationToken)`|Subscribes the client to topics.|Does not apply here.|
|`Task<MqttClientUnsubscribeResult> UnsubscribeAsync(MqttClientUnsubscribeOptions options, CancellationToken cancellationToken)`|Unsubscribes the client from topics.|Does not apply here.|

# ManagedClient
## ManagedClient options

|Method name|Description|Default value|
|-|-|-|
|`Build()`|Builds the options class.|Does not apply here.|
|`WithAutoReconnectDelay(TimeSpan value)`|Sets the auto-reconnect delay.|`TimeSpan.FromSeconds(5.0)`|
|`WithClientOptions(IMqttClientOptions value)`|Tells the managed client to use the configuration available under [Client options](https://github.com/chkr1011/MQTTnet/wiki/Available-options#client-options)|`null`|
|`WithClientOptions(MqttClientOptionsBuilder builder)`|Tells the managed client to use the configuration available under [Client options](https://github.com/chkr1011/MQTTnet/wiki/Available-options#client-options)|`null`|
|`WithClientOptions(Action<MqttClientOptionsBuilder> options)`|Tells the managed client to use the configuration available under [Client options](https://github.com/chkr1011/MQTTnet/wiki/Available-options#client-options)|`null`|
|`WithMaxPendingMessages(int value)`|Tells the client to use the number as maximum amount of pending messages.|`int.MaxValue`|
|`WithPendingMessagesOverflowStrategy(MqttPendingMessagesOverflowStrategy value)`|Tells the client which message overflow strategy should be used.|`MqttPendingMessagesOverflowStrategy.DropNewMessage`|
|`WithStorage(IManagedMqttClientStorage value)`|Tells the client which storage to use.|`null`|

## Managed client methods and properties

|Method name|Description|Default value|
|-|-|-|
|`IApplicationMessageProcessedHandler ApplicationMessageProcessedHandler`|The application message processed handler to perform actions when an application message is processed.|`null`|
|`IApplicationMessageSkippedHandler ApplicationMessageSkippedHandler`|The application message skipped handler to perform actions when an application message is skipped.|`null`|
|`IMqttClientConnectedHandler ConnectedHandler`|The connected handler to perform actions when the connection is established.|`null`|
|`IConnectingFailedHandler ConnectingFailedHandler`|The connecting failed handler to perform actions when the connection failed to be established.|`null`|
|`IMqttClientDisconnectedHandler DisconnectedHandler`|The disconnected handler to perform actions when the connection is lost.|`null`|
|`bool IsConnected`|A value indicating whether the client is connected or not.|`false`|
|`bool IsStarted`|A value indicating whether the client is started or not.|`false`|
|`IManagedMqttClientOptions Options`|The managed client options set to the client.|`null`|
|`int PendingApplicationMessagesCount`|A value that shows the amount of pending application messages.|`0`|
|`ISynchronizingSubscriptionsFailedHandler SynchronizingSubscriptionsFailedHandler`|The synchronizing subscriptions failed handler to perform actions when the synchronization of subscriptions failed.|`null`|
|`Task PublishAsync(ManagedMqttApplicationMessage applicationMessages)`|Publishes a message to a topic.|Does not apply here.|
|`Task StartAsync(IManagedMqttClientOptions options)`|A method to start the client.|Does not apply here.|
|`Task StopAsync()`|A method to stop the client.|Does not apply here.|
|`Task SubscribeAsync(IEnumerable<TopicFilter> topicFilters)`|Subscribes the client to topics.|Does not apply here.|
|`Task UnsubscribeAsync(IEnumerable<string> topics)`|Unsubscribes the client from topics.|Does not apply here.|

# Server

## Server options

|Method name|Description|Default value|
|-|-|-|
|`Build()`|Builds the options class.|Does not apply here.|
|WithApplicationMessageInterceptor(IMqttServerApplicationMessageInterceptor value)|Allows to work with all published messaged from the clients.|`null`|
|WithApplicationMessageInterceptor(Action<MqttApplicationMessageInterceptorContext> value)|Allows to work with all published messaged from the clients.|`null`|
|WithClientId(string value)|Sets the client id used for the server.|`Guid.NewGuid().ToString("N")`|
|WithConnectionBacklog(int value)|Sets the number of connections to keep.|`10`|
|WithConnectionValidator(IMqttServerConnectionValidator value)|Allows to validate connections with a custom handler.|`null`|
|WithConnectionValidator(Action<MqttConnectionValidatorContext> value)|Allows to validate connections with a custom handler.|`null`|
|WithDefaultCommunicationTimeout(TimeSpan value)|Tells the server to use the default communication timeout.|`TimeSpan.FromSeconds(15.0)`|
|WithDefaultEndpoint()|Tells the server to use the default endpoint.|`0.0.0.0:1883`|
|WithDefaultEndpointBoundIPAddress(IPAddress value)|Tells the server to use the default endpoint IPv4 address.|`IPAddress.Any`|
|WithDefaultEndpointBoundIPV6Address(IPAddress value)Tells the server to use the default endpoint IPv6 address.|`IPAddress.IPv6Any`|
|WithDefaultEndpointPort(int value)|Tells the server to use the default endpoint port.|`1883`|
|WithEncryptedEndpoint()|Tells the server to use the encrypted endpoint.|Does not apply here.|
|WithEncryptedEndpointBoundIPAddress(IPAddress value)|Tells the server to use the encrypted endpoint IPv4 address.|`IPAddress.Any`|
|WithEncryptedEndpointBoundIPV6Address(IPAddress value)|Tells the server to use the encrypted endpoint IPv6 address.|`IPAddress.IPv6Any`|
|WithEncryptedEndpointPort(int value)|Tells the server to use the encrypted endpoint port.|`8883`|
|WithEncryptionCertificate(byte[] value)|Tells the server to use the certificate for SSL connections.|`null`|
|WithEncryptionSslProtocol(SslProtocols value)|Tells the server to use the SSL protocol level.|`SslProtocols.Tls12`|
|WithMaxPendingMessagesPerClient(int value)|Tells the server to allow a maximum of pending messages per client.|`250`|
|WithPersistentSessions()|Tells the server to persist sessions.|Does not apply here.|
|WithStorage(IMqttServerStorage value)|Tells the server to use a storage.|`null`|
|WithSubscriptionInterceptor(IMqttServerSubscriptionInterceptor value)|Allows to work with all subscriptions from the clients.|`null`|
|WithSubscriptionInterceptor(Action<MqttSubscriptionInterceptorContext> value)|Allows to work with all subscriptions from the clients.|`null`|
|WithoutDefaultEndpoint()|Disables the default (non SSL) endpoint.|Does not apply here.|
|WithoutEncryptedEndpoint()|Disables the default (SSL) endpoint.|Does not apply here.|

## Server methods and properties

|Method name|Description|Default value|
|-|-|-|
|IMqttServerClientConnectedHandler ClientConnectedHandler|`The connected handler to perform actions when a client connected.`|`null`|
|IMqttServerClientDisconnectedHandler ClientDisconnectedHandler|`The disconnected handler to perform actions when a client lost the connection.`|`null`|
|IMqttServerClientSubscribedTopicHandler ClientSubscribedTopicHandler|`The subscribed handler to perform actions when a client subscribed.`|`null`|
|IMqttServerClientUnsubscribedTopicHandler ClientUnsubscribedTopicHandler|The subscribed handler to perform actions when a client unsubscribed.|`null`|
|IMqttServerOptions Options|The server options set to the client.|`null`|
|IMqttServerStartedHandler StartedHandler|`The started handler to perform actions when the server started.`|`null`|
|IMqttServerStoppedHandler StoppedHandler|`The stopped handler to perform actions when the server stopped.`|`null`|
|Task ClearRetainedApplicationMessagesAsync()|Clears the retained application messages.|Does not apply here.|
|Task<IList<IMqttClientStatus>> GetClientStatusAsync()|Gets the client status.|Does not apply here.|
|Task<IList<MqttApplicationMessage>> GetRetainedApplicationMessagesAsync()|Gets the retained application messages.|Does not apply here.|
|Task<IList<IMqttSessionStatus>> GetSessionStatusAsync()|Gets the session status.|Does not apply here.|
|Task StartAsync(IMqttServerOptions options)|Starts the server.|Does not apply here.|
|Task StopAsync()|Stops the server.|Does not apply here.|
|Task SubscribeAsync(string clientId, ICollection<TopicFilter> topicFilters)|Subscribes the server to topics.|Does not apply here.|
|Task UnsubscribeAsync(string clientId, ICollection<string> topicFilters)|Unsubscribes the server from topics|Does not apply here.|