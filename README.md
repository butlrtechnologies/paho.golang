Eclipse Paho MQTT Go client
===========================

This repository contains the source code for the [Eclipse Paho](http://eclipse.org/paho) MQTT V5 Go client library. 

**Warning breaking change** - Release 0.12 contains a breaking change; see the [release notes](https://github.com/eclipse/paho.golang/releases/tag/v0.12.0). 

Following the release of v0.12, major changes have been introduced to the library ([full QOS1/2 support](https://github.com/eclipse/paho.golang/issues/25)). 
Due to the extent of these changes, it's likely that some users will encounter breaking changes and bugs may have been introduced (managing the session state is quite complex!). 
Please assist us in testing @master (and post your experiences to [this issue](https://github.com/eclipse/paho.golang/issues/207)); the change should enable more people to migrate from the v3 client and gets us a lot closer to V1.0! (see notes at the end of this readme).

There is also a [v3 client](https://github.com/eclipse/paho.mqtt.golang) available (note that this is an older project, and its API is very different to this one).

Installation and Build
----------------------

This client is designed to work with the standard Go tools, so installation is as easy as:

```bash
go get github.com/eclipse/paho.golang
```

Folder Structure
----------------

The main library is in the `paho` folder (so for general usage `import "github.com/eclipse/paho.golang/paho"`). There are 
examples off this folder in `paho/cmd` and extensions in `paho/extensions`.

`autopaho` (`import "github.com/eclipse/paho.golang/autopaho"`) is a fairly simple wrapper that automates the connection 
process and will automatically reconnect should the connection drop. For many users this package will provide a simple 
way to connect and publish/subscribe as well as demonstrating how to use the `paho.golang/paho`.
`autopaho/examples/docker` provides a full example using docker to run a publisher and subscriber (connecting to 
mosquitto).


Reporting bugs
--------------

Please report bugs by raising issues for this project in GitHub [https://github.com/eclipse/paho.golang/issues](https://github.com/eclipse/paho.golang/issues).

A limited number of contributors monitor the issues section so, if you have a general question, please see the
resources in the [more information](#more-information) section for help.

We welcome bug reports, but it is important they are actionable. If we cannot replicate the problem, then it is unlikely 
we will be able to fix it. The information required will vary from issue to issue, but almost all bug reports would be 
expected to include:

* Which version of the package you are using (tag or commit - this should be in your `go.mod` file)
* A full, clear, description of the problem (detail what you are expecting vs what actually happens).
* Configuration information (code showing how you connect, please include all references to `ClientOption`)
* Server details (name and version - e.g. Mosquitto v2.0.18).

If at all possible, please also include:
* Details of your attempts to resolve the issue (what have you tried, what worked, what did not).
* A [minimal, reproducible example](https://stackoverflow.com/help/minimal-reproducible-example). Providing an example
  is the best way to demonstrate the issue you are facing; it is important this includes all relevant information
  (including server configuration). Docker (see `autopaho/examples/docker`) makes it relatively simple to provide a 
  working end-to-end example.
* Server logs covering the period the issue occurred.
* Application Logs (enable debug logging in the library) covering the period the issue occurred. Unless you have isolated 
  the root cause of the issue, please include a link to a full log (including data from well before the problem arose).

It is important to remember that this library does not stand alone; it communicates with a server and any issues you are
seeing may be due to:

* Bugs in your code.
* Bugs in this library.
* The server configuration.
* Bugs in the server.
* Issues with whatever you are communicating with.

When submitting an issue, please ensure that you provide sufficient details to enable us to eliminate causes outside of
this library (e.g. show that a tool like [`mosquitto_pub`](https://mosquitto.org/man/mosquitto_pub-1.html) works).

Contributing
------------

We welcome pull requests, but before your contribution can be accepted by the project, you need to create and
electronically sign the Eclipse Contributor Agreement (ECA) and sign off on the Eclipse Foundation Certificate of Origin.
More information is available in the
[Eclipse Development Resources](http://wiki.eclipse.org/Development_Resources/Contributing_via_Git).

Please raise an issue prior to implementing any major changes; it's better to check that the change is likely to be 
accepted, and discuss the design, before investing your time in it.

More information
----------------

This client aims to implement the [MQTT Version 5.,0 Specification](https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html);
so, if you have questions about the protocol itself, then the spec is a good place to start.

* [Stack Overflow](https://stackoverflow.com/questions/tagged/mqtt) is probably the fastest way to get an answer (but 
is not a discussion forum, so invest the time to ask a [good question](https://stackoverflow.com/help/how-to-ask), and
remember to search existing questions first).
* There is an [MQTT Google Group](https://groups.google.com/forum/?hl=en-US&fromgroups#!forum/mqtt) for general questions 
about the MQTT protocol.
* [Reddit](https://www.reddit.com/r/MQTT/) has a less active MQTT forum but is a good option for open-ended questions.
* `#MQTT` in the [Gophers slack](https://gophers.slack.com/join/shared_invite/zt-1vukscera-OjamkAvBRDw~qgPh~q~cxQ) is 
pretty quiet but questions are generally answered quickly.
* Discussion of the Paho clients takes place on the [Eclipse paho-dev mailing list](https://dev.eclipse.org/mailman/listinfo/paho-dev).

There is much more information available via the [MQTT community site](http://mqtt.org).

QOS1/QOS2 Implementation
----------------

The major feature missing from this library, as at release 0.12, was support for [session persistence](https://github.com/eclipse/paho.golang/issues/25); 
the library effectively operated at QOS0 (QOS1/2 appeared to work, but the delivery guarantees were not honored).

This has now been rectified (in @master); a major change (which, despite testing, is likely to introduce issues!). 

There are still a few TODOs in the code; these flag areas that may require further work (releasing this before 
resolving them all because this is a huge change already).

Please assist us in testing this new code, we are aiming for a release before the end of the year.

## Breaking changes:

* `paho` `ClientOptions.MIDs` has been removed. While it was possible to implement your own MIDService, I suspect that
  no one has done so.
* `paho.Publish` when publishing at QOS1/2 the packet identifier (if acquired) was released if the context expired
  regardless of whether the message had been sent (potentially leading to reuse of the ID and in breach of the spec). 
  This has been changed such that once transmitted, the message will be acknowledged regardless of the publish context
  (but the Publish function will only block until the context expires). The Errors returned now better indicate what occurred.
* `autopaho` CleanSession flag. Previously the CleanSession was hardcoded to `true`; this is no longer the case and
  the default is `false`. Whilst his is potentially a breaking change, `SessionExpiryInterval` will default to 0 meaning
  the session will be removed when the connection drops. As a result this change should have no impact on most users; it
  may be a problem if another application has connected with `SessionExpiryInterval>0` meaning a session exists.

## Known Issues

### Queued messages using Aliases

Topic aliases are not part of the session state. This means that if messages using a topic alias are queued when the
connection drops and then sent when it comes up will not have the desired impact. Possible workaround would be to detect
these and cancel them all when the connection drops.

### Multiple Servers

If a Client may connect to more than one server, or the same server with different ClientIDs, then the user will need to 
carefully manage the store (because each store is specific to one server/ClientID). 

autopaho accepts a slice of servers; if using `CleanStart=false` then these servers MUST be part of a cluster with shared
session state (otherwise messages will be lost).

### SessionExpiryInterval

The client effectively ignores the [Session Expiry Interval](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_Toc3901048) 
when it comes to managing state. This is unlikely to be a problem for most users because the servers `CONNACK` will 
include the Session Present flag, which will inform us if the session has expired (and local state will be removed at 
that time). 

Users may wish to clear session information to save on storage, but this is not something the library currently supports.

### Inflight Message tracking

MQTT v5 allows both the client and server to specify how many simultaneous inflight messages they permit. This is an
excellent addition to the protocol because it improves in-order delivery and can help avoid saturating network links.

This client does not enforce the [Receive Maximum](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_Toc3901049)
for messages being received from the server. This is unlikely to be an issue for most users because most servers should 
honour the limit (if we were checking for this situation, we would need to drop the connection if it was detected).

The client does honor the [Receive Maximum](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_Toc3901083) 
received from the server (indicating how many inflight publishes the client can initiate to the server). However, there
is a limitation; consider the following situation:

1. Connect and server advises receive maximum is 20
2. Publish 20 QOS2 messages 
3. Connection drops before any messages are acknowledged (so there are 20 messages in the session state)
4. Reconnect and server advises receive maximum is 10

In this case the client will exceed receive maximum when retransmitting the messages (I'm have not seen this actually 
happen!). The client should delay resending until there are slots available (but note that inflight messages may still 
exceed receive maximum). This is fixable, but requires some thought and a review of 
[the spec](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_Toc3901251). 

### ACK() Unpredicatable results if called after connection loss

Calling `Client.Ack()` after the connection is closed may have unpredictable results (particularly if the sessionState
is being accessed by a new connection). See issue #160.

---
## AutoReconnect Feature
The AutoReconnectConfig is a new feature introduced to enhance the resilience and stability of MQTT connections. It allows your application to automatically attempt to reconnect to the MQTT broker in the event of a connection loss. This feature is especially useful in environments where network instability or intermittent disconnections are common.

### Configuration
To use the AutoReconnectConfig, include it in your paho.ClientConfig during client initialization. The key parameters of AutoReconnectConfig include:

* `MaxRetries`: The maximum number of reconnection attempts. Set this to -1 for infinite retries.
* `RetryInterval`: The initial delay before the first retry attempt, which increases exponentially on subsequent retries.
* `MaxRetryInterval`: The maximum interval between retry attempts, to prevent excessively long wait times.
* `BackoffFactor`: The factor by which the retry interval increases after each failed attempt.
* `ReconnectHandler`: A function that provides a new connection and connection configuration. This handler is invoked for each reconnection attempt.

### Example Usage
In the provided example, the `AutoReconnectConfig` is configured for an MQTT client. The client attempts to connect to an MQTT broker using TCP with a specified broker address, username, and password. The ReconnectHandler is set up to create a new connection and configure it with the necessary credentials each time a reconnection is attempted.

The client subscribes to a specific MQTT topic, and incoming messages are handled by the registered router handler. The AutoReconnectConfig ensures that the client remains connected or automatically attempts to reconnect in case the connection is lost.

### Handling Connection Loss
With `AutoReconnectConfig`, the client will automatically handle connection losses. If the connection to the MQTT broker is disrupted, the client will initiate reconnection attempts based on the configured parameters. This process is transparent to the application, reducing the need for manual intervention and improving the overall reliability of your MQTT communication.
```go
mqtt_url := "localhost"
mqtt_username := "username"
mqtt_password := "password"
topic := "test/topic"

broker := fmt.Sprintf("%s:%d", mqtt_url, 1883)
conn, err := net.Dial("tcp", broker)
if err != nil {
  return err
}
clientid := "example_" + ksuid.New().String()

config := paho.ClientConfig{
  Conn: conn,
  AutoReconnectConfig: &paho.ReconnectConfig{
    MaxRetries:       -1,
    RetryInterval:    1 * time.Second,
    MaxRetryInterval: 60 * time.Second,
    BackoffFactor:    2,
    ReconnectHandler: func() (net.Conn, *paho.Connect, error) {
      conn, err := net.Dial("tcp", broker)
      if err != nil {
        return nil, nil, err
      }

      return conn, &paho.Connect{
        KeepAlive:    30,
        Username:     mqtt_username,
        Password:     []byte(mqtt_password),
        UsernameFlag: true,
        PasswordFlag: true,
        ClientID:     clientid,
      }, nil
    },
  },
  Router: paho.NewStandardRouter(),
}

config.Router.RegisterHandler(topic, func(msg *paho.Publish) {
  // handle incoming message
})

c := paho.NewClient(config)
ca, err := c.Connect(context.Background(), &paho.Connect{
  KeepAlive:    30,
  Username:     mqtt_username,
  Password:     []byte(mqtt_password),
  UsernameFlag: true,
  PasswordFlag: true,
  ClientID:     clientid,
})
if err != nil {
  return err
}
if ca.ReasonCode != 0 {
  return fmt.Errorf("failed to connect to %s : %v - %s", broker, ca.ReasonCode, ca.Properties.ReasonString)
}

_, err = c.Subscribe(context.Background(), &paho.Subscribe{
  Subscriptions: []paho.SubscribeOptions{{
    Topic: topic,
    QoS:   0,
  }},
})
if err != nil {
  return err
}
```