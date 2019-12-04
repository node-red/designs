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

**By default, a Node cannot be timed out**

It will be up to individual implementations to decide if it makes sense to add
timeout handling. Some nodes, such as the Change and Switch nodes do not perform
any long-running activity, so would not make sense to enable timeout handling in
them.

To enable timeout handling a node must add an event handler for the `timeout` event:

    this.on('timeout', function(msg) {
        // Process this timeout. For example, cancel any in-flight work
    })

By adding this handler, the node is telling the runtime it can be timed out and,
therefore, that the node *will* call `node.done(msg)` when it finishes with each
message.

The runtime will track the messages passed to the node using a `Set` - this
removes the need to rely on any particular message property value (such as `_msgId`)
that could be modified by the node.


If `node.done()` is not called within the required timeout the runtime will call
`Node.error(err,msg)` and trigger the timeout listener if one is registered.

### Next steps

Some implementation considerations and outstanding questions:

 - how should a Function node indicate it can be timed-out? It could be able to set
   a timeout handler in the same was as described above - but that would be
   evaluated with every single message the Function receives.

 - This api assumes a default timeout value is applied to all enabled nodes.
    - Should it be possible for individual nodes to be given a custom timeout value?
    - If so, how does the user provide that value? The Editor does not know if the
      runtime instance has added a `timeout` handler
      - add another flag in the node's HTML file?
      - add the option in the UI regardless?
    - An node-level api will be needed for the node to give its value to override
      the default - `node.setTimeout(secs)`.

 - The simple implementation of a timeout will be to add a `setTimeout` for every
   message that is passed to the node - clearing the timeout when `done` is called.
   That does, however, lead to a performance issue if we are creating and
   clearing hundreds of timers a second. A more careful design would be to have
   one `setTimeout` active per node that checks for the next thing to timeout.
   This is probably the most critical part of the implementation to get right.

## History

  - 2019-03-26 - split out from Messaging API design
