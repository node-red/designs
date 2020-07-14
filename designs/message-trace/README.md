---
state: draft 
---

# Message Tracing by metric log 

<!--This is the title of the proposal. It should communicate the key purpose of the
proposal and will be how it is referred to in general discussion.-->

## Summary

<!--This section provides the high-level description of the feature being proposed.
It should provide enough information for end-users to understand what the feature
is trying to achieve.-->
To enable precise tracing of each message, we propose to make the runtime emit
log message whenever the message processing is finished.   For more complex
tracing for nodes which splitting or joining the messages, these nodes may
emit a *correlation* log to provide information of
which message is depend on another message.

## Authors

 - @k-toumura

## Details

### Use case

In debugging, we want to trace the messages. Currently, we use Debug nodes to assure a process in some node is finished. But:

- we can't know the message is received and start processing in some node.
- inserting Debug nodes is cumbersome process, and readability of process become worse.

By enabling metric log, we can see message IDs that is sent/received.
But, if the node doesn't send a message, we can't assure that
the processing in the node is finished.

Moreover, if the node send a message asynchronously, it is difficult
to analyze a causal relationship between messages.

For example, there is a flow configured like following diagram:
![sample flow](sampleflow.png)
and the runtime emittted the following metric logs.
![sample log](samplelog.png)

Node "D" receives message "X", run join process, and then emit message to node "E",
but, we can't assure that messages "Y" and "Z" been processed, or still queued in node "D".

### Requirement

To trace a message, we should know:
- when the processing of the message is finished.
- which messages are related to the message. 

### Proposed changes

#### Metric log for message process completion in a node

Emit metric log when nodes calls `done()` callback, such like:
```
26 Feb 09:33:26 - [metric] {"level":99,"nodeid":"8b1f2328.8e972","event":"node.join.done","msgid":"2522bce.e667944","timestamp":1582677206863}

```
where:
 - nodeid: id of a node
 - event: `node.NODETYPE.done`
 - msgid: id of a message that the node has finished processing
 - timestamp: Unix time in millisecond

#### Metric log to correlate between messages

Emit metric log when sent a message which is related to other messages using:
```
node.metric("correlate", msg, relatedMsgIds)
```
where:
- msg: message in process.  For example, when join node send a concatenated message, use the message as an argument of `node.metric()`.
- relatedMsgIds: array of ids of related messages.

Note: we can put more complex information on the third argument of `node.metric()`.  For example, we can make it a object which contains a array of message ids and a relation type such as `"isPartOf"`, `"splitInto"`, `"isDependsOn"`, etc.

### Usage example

For the example shown in Use cases, the proposed metric logging makes message traceable.  

![Correlation](correlationlog.png)

- by `node.join.correlate` event, we can assume that message X is related to message Y and Z.
- by `node.join.done` event, we can assume that processing of message X, Y and Z are completed.

## Related topics

- [Metrics Usage design note](../metrics-usage.md)
- [Graceful Shutdown design note](../gracerul-shutdown/README.md)

## History

- 2020-07-14 - Initial proposal submitted

