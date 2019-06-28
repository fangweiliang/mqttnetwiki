# MQTT topics

## Explanation
MQTT topics are used on the broker (the server) to filter messages for connected clients. Topics may consist of one to multiple levels. Levels in the topic are separated by a forward slash (`/`).

Clients do not need to _create_ the topics before they publish to it. The broker accepts valid topics without initialization. Of course, brokers can restrict topics, if they like.

Each topic must have at least 1 character and at most 65535 bytes. Topics are case-sensitive. The forward slash alone is a valid topic. It is allowed to include any UTF-8 string (even non printable ones and the empty char or space char). Topics must not include the null character (Unicode U+0000).

## Wildcards
Clients can subscribe to the exact topic or to a wildcard to subscribe to mutilple topics simultaneously. Wildcards can only be used to subscribe to topics, not to publish a message. There are two different kinds of wildcards:

### Single level wildcard: `+`
A single-level wildcard replaces one topic level. The `+` symbol represents a single-level wildcard in a topic.

E.g. `something/+/somethingElse` will match e.g. `something/abc/somethingElse`. but not `something/abc/defg/somethingElse`.

The `+` wildcard can be used multiple times in a topic string, including multiple `+` separated by a `/`:

* `a/+/+/+/nvbshe` is valid
* `a/+/+/` is valid
* `+/+/+` is valid

### Multi level wildcard: `#`
The multi-level wildcard covers many topic levels. The `#`symbol represents the multi-level wild card. The `#` wildcard **must not be used more often than once** and **only at the end of the topic**.

When a client subscribes to a topic with a multi-level wildcard, it receives all messages of a topic that begins with the pattern before the wildcard character, no matter how long or deep the topic is. If you specify only the multi-level wildcard as a topic (`#`), you receive all messages that are sent to the MQTT broker.

E.g. `sensor/data/foo/#` would match `sensor/data/foo/abc/a13re/adff` and also `sensor/data/foo/abc`.

Invalid topic filter examples with the `#` wildcard would be e.g.:

* `#/a`
* `a/a/#/a/#`

### Combination of wildcards
Wildcards can be combined. E.g. `sensor/+/bar/#` matches `sensor/foo/bar/baz/wibble/json` and `sensor/bar/bar/black/sheep`, but does not match e.g. `sensor/bar/abc/d`.

### Topics beginning with `$`
Generally, you can name your MQTT topics as you wish. However, there is one exception: Topics that start with a `$` are not part of the subscription when you subscribe to the multi-level wildcard as a topic (`#`). The `$` symbol topics are reserved for internal statistics of the MQTT broker. Clients cannot publish messages to these topics.

At the moment, there is no official standardization for such topics. Commonly, `$SYS/` is used for all the following information, but broker implementations varies. Here are some examples:
* `$SYS/broker/clients/connected`
* `$SYS/broker/clients/disconnected`
* `$SYS/broker/clients/total`
* `$SYS/broker/messages/sent`
* `$SYS/broker/uptime`

## Best practices
* Never use a leading forward slash. E.g. `/a/b`: The leading forward slash introduces an unnecessary topic level with a zero character at the front!
* Never use spaces or non-printable chars in a topic. Generally, use ASCII-only chars. E.g. `/a/ a`: This makes topics nearly impossible to read for programmers and users.
* Keep the topic short and concise. When it comes to small devices, every byte counts and topic length has a big impact.
* Embed a unique identifier or the Client Id into the topic. It can be very helpful to include the unique identifier of the publishing client in the topic. The unique identifier in the topic helps you identify who sent the message.

## Sources
* https://stackoverflow.com/questions/56796583/are-multiple-allowed-in-mqtt-topics#
* https://www.hivemq.com/blog/mqtt-essentials-part-5-mqtt-topics-best-practices/
* https://mosquitto.org/man/mqtt-7.html
* http://mqtt.org/
* http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html