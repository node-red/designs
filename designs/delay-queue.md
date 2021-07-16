---
state: draft
---

# Add Queuing options to Delay node

## Summary

This design looks at how to add some more control to the basic queuing options in the delay node.
There are several contrib queuing nodes available, but nothing in the core. As queuing of messages is a relatively common requirement this design looks at adding queuing to the existing delay node as that already has a storage buffer, with capacity limit handling, and done() capability.

## Authors

 - @ceejay

## Introduction

The original Delay node has two main modes of operation - Delay and Rate Limit.

When in Delay mode any message arriving is pushed to an array created with a timeout based on other configuration parameters (fixed, random, set by msg.delay).

When in rate limit mode messages are pushed to an array, and if not already running a timer loop started. Every tick the message at the front of the array is shifted off and sent.

In both modes
    - if `msg.reset` is present then any existing messages are dropped, timeouts cleared, and arrays emptied. No messages are sent onwards. If payload and reset are present then payload is lost (as nothing is sent)
    - if `msg.flush` is present then all existing messages are sent immediately to the output. (IE the timers are triggered). If payload and flush are present then payload is added to queue and sent as part of flush. (No data lost).

## Details

This design is to extend the basic use of the existing delay node to allow it to be used as a more controllable queue.

    - The ability to "take" the next message. IE a downstream process could send a control message to release the next message for processing (rather than having to wait). This way more variable timings can be accomodated rather than having to set the timer to a worst case scenario.
    - Ability to "take" a batch of messages. Likewise the ability to release a controlled number of messages at one time may provide greater flexibility for downstream processing.
    - Ability to see how deep the queue is. If the status reflected the actual depth of the queue then a process could decide to wait for more to arrive, process a batch, or dump excess messages.
    - Ability to retry sending a message. Many scenarios for queuing are based on "save data while waiting for a connection - when connected send data - if OK great - if fail retry.
    - Creation of Stack type queues - Last In First Out (LIFO).

The existing node only removes things from the array(queue) based on time. The only control the user has is to either reset or flush the entire queue.

### Proposal 1 - PR3059 (merged)

The initial proposal is to add mode control the the existing flush operation. Rather than just being a flag, allow `msg.flush` to accept a numeric value of item to remove from the queue. And at the same time change the node-status to reflect the depth of the queue. This allows several new scenarios to be relisable - namely the ability to feedback into the node and release the next message(s), and monitor the depth of the queue.

### Proposal 2 - PR3069

As the current rate limit function is based around a simple array the obvious operations are push, pop, shift and unshift. The current mode uses push to add to the end of the queue, and shift to remove items from the front of the queue.

The node could be expanded to allow access to the other methods such as unshift to force an item to the head of the queue. This would then let a process create a retry by pushing any failed message back into the queue - and if combined with the new flush:1 ability from PR3059 would retry immediately.

Pushing to the front of the queue would also allow the creation of LIFO (Last In First Out) type queues ie a stack... (vs the existing FIFO style).

For this I propose adding a property `msg.lifo` a boolean flag that if true would unshift the corresponding message to the front of the array.

*Note*: This flag makes no sense for other modes of operation - standard delays are all just timer based so you can (if allowed) set the msg.rate to be really short to make it come out more quickly. And in rate limit mode if you select drop interediate messages then by default no other messages are accepted until the next time slot. (The queue is never more than 1 deep).

### Other possibilities include

    - allow a TTL (time to live) for a message in the queue - if it expires the message removes itself from the queue.

    - access to array pop capability... not sure of a use case apart from to remove from end of queue in which case why not use split.

    - access to array split - to remove items from the queue - but how would user know which to remove - again I can't see a use case.

    - ability to not set timers. Currently all the above still set the timer as a fundamental property. I think this can be justified in that it can act as a watchdog to timeout and release items if the user does not "pull" them from the queue first.
    But there could be a version - a "static" queue - where the timer is not set at all, which then relies on the user and external flow to ensure messages are handled in a timely manner. If so this would require UI changes (unless we allow time of 0 = infinite/no timer) in which case does queuing become a completely different mode of operaton of the node ? and if it did then would we need to add back in optional timeouts to that mode.

    - ability to pause/restart queue (HARDER) - currently implementation is an array of timers. If this was considered necessary then the setTimeout/setInterval function would need to be extended to be able to pause all of them, and restart all of them.


## History

  - 2021-07-09 - Initial Draft
