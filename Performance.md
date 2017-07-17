Whilst dbus-broker is still a young project, and no serious performance optimizations have been performed, this page outlines some of the challenges and design-decisions that are relevant to any performance analysis.

## Pipelining

Each time we call into the kernel, we attempt to read or write as much data as possible (up to some fixed limits), in particular, we do not limit ourselves to reading/writing one message at a time.

A nice example of why this may be beneficial is that a client could connect, push its entire SASL exchange, the initial `Hello()` message, and perhaps a call to `RequestName()`, the broker would read all of this data from the kernel with one call, process it all in the same main-loop iteration, and write out all the responses with one more call to the kernel. Assuming it is the length of the pending dispatch-queue that is the limiting factor on a busy bus, being able to do connect+request+reply in one main-loop iteration (rather than six, or even twelve) should improve latency accordingly, in the worst-case.

## Lookup maps

All lookups are done in `log(n)` time, and maps are kept as small and local as possible.

By way of example, pending replies are indexed per (sending) peer in an rbtree, so the cost of looking up the destination of a pending reply is `log(n)` in the number of outstanding replies from the given peer (as opposed to, say, `n` in the number of pending replies on the entire bus, which at some point caused issues for the reference implementation).

## Matches

A major source of uncontrollable performance is matches. In principle, every message sent on the bus must be compared to every match to determine who should receive it. As any peer must be able to install (relatively many) matches, this intrinsically linear operation may cause unexpected slowdowns.

At the moment we index matches by the `sender` field, so at least a sender will only look at matches that has been installed on one of its names (in particular, the matches follow the names, so no extra work must be done when names are moved from one peer to another). A caveat with this is that it is possible to have matches without a sender specified, so we also must keep a global list of wildcard matches that all messages must be compared against. The use of such matches should hence be discouraged, but due to backwards compatibility, they cannot be banned outright.

A future optimization of matches is to probabilistically reduce the cost of comparison from `n` to `log(n)` in the number of false positives in the common case (matching will necessarily always be linear in true positives).

## Policy

Due to the nature of the D-Bus policy, policy application to each transferred message is, like matches, too intrinsically linear. Like matches we index policies by the `send_destination` (resp., `receve_sender`) fields. This does not change the algorithmic performance, but it greatly reduces the number of false positives we have to compare on each message transaction.

In the future, we hope to introduce a new policy language, which is both much more intuitive to write/read and also can be implemented with much more reasonable performance cost.