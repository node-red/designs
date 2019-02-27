---
state: draft
---

# Dynamic MQTT Node

## Summary

There is a requirement for the MQTT nodes to allow for more dynamic configuration.
There are two uses:

 - determine a subscription topic at runtime
 - modify broker connection details at runtime

This design note proposes a new MQTT node that can be used to satisfy these requirements.

## Authors

 - @knolleary

## Details

Here's the basic outline:

 - A new MQTT node is proposed called `mqtt-control` (name, as ever, subject to change).
 - The node has an input and an output so sits in the middle of a flow
 - The node is configured with an `mqtt-broker` node - this broker node provides the connection used by the control node.
 - The node can be passed messages with a well-defined property (eg `msg.mqttControl`) that causes it to take an action.
 - The actions the node can take are:
    - Connect to its broker (with any of the `mqtt-broker` node's properties overridden by properties in the message)
    - Disconnect from the broker
    - Subscribe to a topic
    - Unsubscribe from a topic
    - Publish a message to a topic
 - The node will send any message it receives due to its subscriptions.
 - It will also pass through messages it receives when the action is complete. **TODO**: identify how to distinguish these from subscription messages


Some observations from this design:

 - the control node is controlling the `mqtt-broker` node. Other regular nodes can also use that `mqtt-broker` node so will also be effected by changes.
 - the `mqtt-broker` config node will have a new option to not auto-connect. This would be used where the decision to connect is left to the application's use of the control node.

---

### `msg.mqttControl`

This message property is used to control the `mqtt-control` node. It's content determines the action the control node will take.

#### Connect

The connect action will connect the mqtt-control node if it is not currently connected.

If the broker is already connected, the action will be ignored. (**Q**: or should it cause a disconnect and reconnect?)

The `broker` object can contain any property to override the mqtt-broker node.

~~~
msg.mqttControl = {
   "action": "connect",
   "broker": {
      "url": "mqtt://localhost:1883",
      "clientid": "my-client-id",
      "keepalive": 60,
      "clean": true
      "username": "",
      "password": "",
      "birth": {
         "topic":"",
         "qos": "",
         "retain": false,
         "payload": ""
      },
      "close": {
         "topic":"",
         "qos": "",
         "retain": false,
         "payload": ""
      },
      "will": {
         "topic":"",
         "qos": "",
         "retain": false,
         "payload": ""
      }
   }
}
~~~

#### Disconnect

~~~
msg.mqttControl = {
   "action": "disconnect"
}
~~~

#### Subscribe

~~~
msg.mqttControl = {
   "action": "subscribe",
   "topic": "a/b/c"
}
~~~

#### Unsubscribe

~~~
msg.mqttControl = {
   "action": "unsubscribe",
   "topic": "a/b/c"
}
~~~

#### Publish

The publish action will use the same standard properties as the existing `mqtt out` node. But the `msg.mqttControl` property *must* be present to tell the control node to publish.
~~~
msg.mqttControl = {
   "action": "publish"
}
~~~



## History

 - 2019-02-25 - migrated from Design note wiki
