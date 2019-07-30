---
state: in-progress
---

# Node Messaging API

## Summary

The current node api doesn't allow the runtime to know when a message has
been fully dealt with. This proposal looks at a new API for nodes that will
address this issue.

## Authors

 - @knolleary

## Details

The current mechanism for a node to handle messages is as follows:

 - a handler is registered for the `input` event
 - within that handler a node can call `this.send()` as many times as it wants
 - if an error is hit, it can call `this.error(err,msg)`

```
var node = this;
this.on('input', function(msg) {
    // do something with 'msg'
    if (!err) {
        node.send(msg);
    } else {
        node.error(err,msg);
    }
});
```

This simple system has a few limitations that we want to address:

 1. we cannot correlate a message being received with one being sent - particularly when there is async code involved in the handler
 2. we do not know if a node has finished processing a received message
 3. due to 1 & 2, we cannot build any timeout feature into the node
 4. we know of a use case where an embedder of node-red requires nodes to be strictly one-in, one-out, and that should be policed by the runtime. Putting aside the specifics, we cannot provide any such mode of operation with the current model

This design note explores how this mechanism could be updated to satisfy these limitations.

### Use Cases

 - Allow a user to create a Flow that is notified when a message is successfully
   processed at a critical point of a flow. For example, the `Email Out` node
   has sent its message.
 - Allow the runtime to track messages through a flow. This can be used to:
   - build a timeout mechanism into nodes
   - allow a flow to be gracefully shutdown - allowing in-progress messages to complete

### `node.on("input", function(msg, send, done) {})`

The callback handler for the `input` event is updated to include `send` and `done`
arguments.

The `send` argument is a function that is equivalent to `node.send`. Using this
function will allow the runtime to correlate the call with the message that
triggered the callback.

The `done` argument is a function that must be called when the node has finished
processing a message.

The `done` function takes one argument - an optional `error`. If this is provided,
the node will log the error as if `node.error(error,msg)` was called. A node should
*not* do both `node.error` and `done(error)` as this will cause the error to be
reported twice.

When `done()` is called, any in-scope `Complete` nodes will be triggered. This is
a new node type added under this proposal.

To ensure backwards compatibility, a node should check if `send` and `done` exist
before using them. That will allow the node to be installed in older versions of Node-RED.

```
var node = this;
this.on('input', function(msg, send, done) {
    send = send || node.send;
    // do something with 'msg'
    if (!err) {
        node.send(msg);
    } else {
        node.error(err,msg);
    }
    if (done) {
        done(msg);
    }
});
```

### Function node

If a Function node returns a message, or array of messages, then `done` will be
called implicitly by the node.

If the Function node does not return any messages then the Function must call `node.done()`.
This allows a Function node that does asynchronous work (optionally using `node.send()`)
to call `node.done()` at the right time.

### `Complete` node

This design has changed quite a few times. It has bounced between adding a new
node to handle the 'done' events, and reusing the `Status` node.

Having modelled the `Status` node approach, a number of issues were identified that
made it less ideal.

 - Existing flows using Status nodes will suddenly start receiving the 'done' status
   events. If the flows are not expecting them, that could have bad side-effects.
 - A workaround to that would be to add an option to the Status node to opt into
   receiving 'done' events. But that gets messy.
 - Overloading the Status event with the Done event also gets messy when there is
   also an error to report.

Having a new node type to handle the 'done' events is the cleanest way for a flow
author to create a flow that can react to the different types of event - status,
error and done.

This node will be very similar in design to the Catch/Status node. However, it will
*not* offer the default 'Handle all' type of behaviour the others do. The user
*must* target it at specific nodes in the flow. They could chose to select all,
but it would not be a top-level menu option as it is with the Catch/Status nodes.

#### Timeout handling

This is moving to a separate Design note and not part of the delivery of this
design.

[https://github.com/node-red/designs/blob/master/designs/timeout-api.md]()


## History

  - 2019-07-09 - going back to the callback approach
  - 2019-06-19 - another iteration of the design
  - 2019-03-26 - rewritten to cover new design proposal
  - 2019-02-27 - migrated from Design note wiki
