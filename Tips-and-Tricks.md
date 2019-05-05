# Logging
It is recommended to disable logging (Global logger or custom logger) which running in a live environment because it adds a huge overhead to message processing. It should be avoided to attach to the global MQTTnet logger at all because an already attached method (which may not process the entry further on) leads to generation of log messages internally.

# Prefer builders
MQTTnet comes with some builders like the _MqttClientOptionsBuilder_ or _MqttApplicationMessageBuilder_ which are supporting a fluent API. Also these builders are used to achieve version compatibility. It is supported to change the _MqttClientOptions_ class directly but this file may have breaking changes across versions of MQTTnet. The builders instead will usually provide backward compatibility (when possible).

# MQTT best practices
This project only covers the implementation of a basic MQTT library. For best practices, general recommendation ans specifications please visit https://www.hivemq.com/mqtt/.

