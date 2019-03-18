---
state: draft | in-progress | complete
---

> To get started with this template:
>
> 1. **Make a copy of this template.** Copy this template into the `designs`
>   folder and name it `my-title.md`. If the proposal will include images, a folder
>   should be created for the proposal and its images: `my-title/README.md`.
>   
>    Remember to delete this getting started text - but ensure the YAML
>    header above remains.
>
> 2. **Create an initial proposal.** The proposal should set out the high-level
>   goals of the feature with enough detail to review the intent and direction of the
>   feature.
>
> 3. **Create a pull request.** This will be used to drive discussion on the
>   proposal. The goal is to merge proposals early and not try to tackle everything
>   in a single go.
>
> 4. **Develop the design.** The discussion on the design will provide guidance
>   on what further material is needed.

# Options for enhancements to Status node API

How do recent recent updates to status node functionality affect need for node-messaging api ?
Several options exist to clarify.

## Summary

Recent updates to status node mean that it can now pass back more information to the runtime.
Does this go part/most of the way to fixing the node-messaging requirement without requiring an extra node ?
Should the status node API (node.status()) be made to be more in line with the catch node (node.error()) ?

## Authors

 - Dave Conway-Jones

## Details

#### Situation as was (prior to v0.20)

**Status Node call**

    node.status({shape:"ring",color:"red",text:"ooops"})

i.e. a single parameter (object with 3 properties)

**Catch Node call**

    node.error("some error text for sidebar",msg)

i.e. two parameters (string, object that caused the error)

---

There is also, as part of the Node Messaging API work, a proposal for a `success` (or `complete`) node to handle nodes that wish to report success of an operation (eg database insert, file operation, etc), but that do not wish to impinge on the main flow - either as they are at the end of a flow, or they don't wish to add a second output to the node just for this purpose. The proposal adds

    node.complete(msg, null, msg);
    node.complete(msg, err);

There is also a separate proposal to allow for a very simple status messages by allowing a text only payload, e.g.

    node.status("simple status message");

#### Situation as of v0.20

The Status node call can now be passed extra properties that are filtered out before being passed to the UI via the websocket, but remain intact and are now reported by the status node.

    node.status({shape:"dot",color:"green",text:"ok",extra:"Hello",more;"World"})

This opens up the possibility of using this to pass extra information back via the status node. For example we have been asked to provide a way for the email node to report successful delivery of a message, and provide a correlation id so it can be removed from the pending buffer.

The simplest way to do this would be to just return the complete msg object as part of the status. e.g.

    node.status({shape:"dot",color:"green",text:"ok", msg:msg })

**BUT** this now seems to be basically the same idea as the Success node proposal above, but doesn't require a new node. However the call semantics are different from that proposal.

### Options

1. Embrace node.status as-now-is, and formally support adding msg as part of the object that is sent back. This already works today with no further code changes.

2. Change the semantics of the node.status call to match the node.error call, to add an optional second parameter which is the complete msg object eg `node.status({...},msg)` - and ideally to also allow simple string status - which would make it even more similar to node.error.

In either case - does it now mean we could drop the idea of the success node as both could handle the use cases for that, without requiring a new node or too much extra effort ?

To me (DCJ) the second option (plus the addition of string only status capability) would then align it, and make it consistent with, the node.error - so catch and status could be "driven" in the same way, so making the reporting of success more similar to that of failure. But the first option does have the benefit of already working today if documented.


## History

This should be a list of major milestones in the life of the proposal. For example:

- 2019-03-18 - Initial proposal submitted
- yyyy-mm-dd - Moved to in-progress
- yyyy-mm-dd - Shipped in Node-RED 0.20 - moved to complete folder
