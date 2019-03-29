---
state: draft
---

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

    node.done(err, msg);

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

1. Embrace node.status as-now-is, and formally support adding msg as part of the object that is sent back. This already works today with no further code changes. For example

    node.status( {text:"hello", msg:msg} );

2. Change the semantics of the node.status call to match the node.error call, to add an optional second parameter which is the complete msg object eg `node.status({...},msg)` - and ideally to also allow simple string status - which would make it even more similar to node.error. For example

    node.status( {text:"hello"}, msg);


In either case - does it now mean we could drop the idea of the success node as both could handle the use cases for that, without requiring a new node or too much extra effort ?

To me (DCJ) the second option (plus the addition of string only status capability) would then align it, and make it consistent with, the node.error - so catch and status could be "driven" in the same way, so making the reporting of success more similar to that of failure. But the first option does have the benefit of already working today if documented.

#### Examples

    node.status( {text:"hello"} );          
    node.status( "Text only status" );   // new text only mode.   

Exactly the same semantics as today - send the status event to the editor and trigger any in-scope Status nodes.

    node.status( {text:"hello"}, msg );     
    node.status( "simple text", msg );

Send the status event to the editor and trigger any in-scope Status nodes. It will use msg as the message emitted by the Status node - with the `.status` property set. This is consistent with the node.error("error", msg) semantics. Nodes that adopt this will still work on earlier versions of NR as the second argument will be ignored.

    node.status( null, msg );        

If the status part is null is should not be passed to the editor - but should still trigger the status node.




## History

This should be a list of major milestones in the life of the proposal. For example:

- 2019-03-18 - Initial proposal submitted
- yyyy-mm-dd - Moved to in-progress
- yyyy-mm-dd - Shipped in Node-RED 0.20 - moved to complete folder
