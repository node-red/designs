---
state: draft
---

# Pluggable Message Routing

## Summary

The mechanism by which messages are passed from one node to the next should be
a pluggable component of the runtime. This would enable, for example, a flow that
spans multiple runtime instances. Other use cases:

 - flow debugger
 - adding custom low-level logging of node send/receive events including the
   full message data

## Authors

 - @knolleary
 - @mblackstock

## Details

### Use Cases

#### 1. Flows that spanning multiple runtimes

<details>

 - instances running on separate cores routed manually or based on a policy or algorithm.
 - runtimes on separate machines in a cluster
 - runtimes in a fog deployment, e.g. on devices, gateways, cloud routed dynamically
   depending on the mobility of a device, associated connectivity, location, etc.

Note: The method for managing and distributing instances running in different cores
or machines is outside the scope of this design.

</details>

#### 2. Flow debugger

<details>

 - the ability to set breakpoints in a flow that will halt the passing of messages
   and allow a user to inspect the state of the system
 - provide more detailed information about messages passing through the flows
   without having to instrument it with multiple Debug nodes.
</details>

### Concepts

The runtime will maintain a stack of message routers - in a similar style to
how Express allows for a stack of middleware to handle requests.

The stack of routers can be changed without restarting the Node-RED process,
but it will require flows to be restarted.

When the runtime needs to deliver a message between two nodes, it gets passed
to the first router in the stack. The router operates on the message then can
choose whether to pass it to the next router in the stack or stop processing.

This model allows for some routers to act more as observers in the stack, rather
than be actively responsible for transporting messages.

### Router Module API

#### `RouterModule.open(runtime) => Promise`

#### `RouterModule.close(runtime) => Promise`

#### `RouterModule.send( .... , done)`

This will be the function that does the work of routing messages.

_The exact function signature is still to be defined_

The information that needs to be passed into the send function:

 - the source node
 - the destination node - as node id
 - the message to pass

The function needs a way to:

 - pass the msg to the next router in the stack
 - halt processing of the message

Does the router module need a wider view of the deployed flows? If it is meant
to support a distributed model, does it need to know when deploys occur so it
can preconfigure any necessary routes? Or is that done via another channel?



## Reference

 - https://trello.com/c/J7UDbQVP/66-pluggable-message-routing
 - https://github.com/node-red/node-red/wiki/Pluggable-Message-Routing

## History

This should be a list of major milestones in the life of the proposal. For example:

- 2019-03-27 - Initial proposal submitted
