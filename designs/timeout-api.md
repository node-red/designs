---
state: draft
---

# Node Timout API

## Summary

This design looks at how to add a timeout mechanism to nodes. This will allow
flows to be created that can handle hangs or other unexpected events that prevent
the normal operation of a flow.

## Authors

 - @knolleary

## Details

This design builds on the new [Messsaging API design](node-messaging-api) that
introduces the `node.done` function for a node to indicate it has completed
handling a message.

It will be possible for the runtime to timeout a node that does not complete its
processing of a message within a given time. But this can only work if the runtime
knows a node has been updated to call `node.done`.

By default, a Node cannot be timed out.

It will be up to individual implementations to decide if it makes sense to add
timeout handling.

If they do choose to support timeout logic, they will:

1. call `Node.setTimeout(secs)` in their constructor to set their timeout value.
2. optionally add handler for the `timeout` event:

        this.on('timeout', function(msg) {
            // Process this timeout. For example, cancel any in-flight
        })

If the node has set a timeout, then the runtime will track the messages passed
to the node using `_msgid`. If `node.done()` is not called within the required
timeout the runtime will call `Node.error(err,msg)` and trigger the timeout listener
if one is registered. TODO: describe the precise details of the `err` passed in
this case.

### Next steps

Some implementation considerations and outstanding questions:

 - `_msgid` might not be unique - for example a sequence of messages from a `Split`
   node. That makes tracking messages in/out of a node tricky. Should the node
   generate new `_msgid` values if it detects a duplication? That could break
   message tracking in the metrics event layer.

 - how should a Function node indicate it can be timed-out? Adding `node.setTimeout()`
   to the top of the function code would mean it is called after a msg is received - too
   late to start the timeout handling logic. It could be exposed as a config
   option on the node, which leads to the next point....

 - Should it be possible in the UI to set a timeout on any node? Or at least those
   nodes that support a timeout? The question becomes how the editor knows a node
   can be timed-out; the call to `setTimeout()` is in the runtime and only made
   when a Node is being created. Does the node's HTML also need a flag adding
   to say it can be timed out?


## History

  - 2019-03-26 - split out from Messaging API design
