As is traditional on linux, we account resources on a per-UID basis. This is not intrinsic to our implementation, and could easily be changed to something more fine-grained if that proves necessary.

In general, peers connected to the broker are run under a different UID from the broker itself (and from each other), so we must restrict their resource usage to make sure they cannot cause a denial-of-service by consuming more resources than the broker has available, or resources that should be reserved from peers owned by another UID.

To the extent possible, we wish to account resource usage directly on a UID, and not, for instance, in a hierarchy based on peers or similar. The reason for this is that each UID must be able to connect many peers, so if we limit resources per-peer, the limits would have to be artificially low in order to keep the total usage limits per-UID reasonable. Moreover, it makes no sense to try to protect peers owned by the same UID from each other.

There are two ways resources can be consumed: a peer can directly consume its own resources, or a peer can cause another peer to consume its resources. These cases are related, but different. We start out by explaining the accounting in the first case, and later extend it to the second.

## Types of Resources

There are four different categories of resources that are accounted. Each of these has a fixed (but configurable) per-UID limit. It is a configuration error or programmatic error to exceed these quotas. And exceeding them may cause the peer to be disconnected, or an error to be returned, depending on the situation. For details see the page on [reliability](Reliability).

### Memory

The amount of memory consumed by each peer is accounted, and limited. This can be objects the peer owns in the broker, temporary buffers or message payloads.

### File Descriptors

FDs are a precious resource on a linux system, and must be limited explicitly, as the kernel puts a (relatively low) limit on the number of FDs that can be owned by the broker at once.

### Matches

As described on the [performance](Performance) page, matches (potentially) introduce a _linear_ cost to every message transaction. Nothing else on the system has such potentially adverse effects, so we explicitly limit the number of matches to try to keep this under control.

### Objects

There are all sorts of other objects that a peer can own in the broker. Their memory is already accounted, but we also put a cap on the number of objects as the number of objects does introduce a run-time cost as the lookup maps grow (albeit a logarithmic one, so this limit may be a lot larger than the match limit).

## Quotas

As mentioned above, it is possible for one peer to consume resources from another. This happens when a peer queues a message on another, the resources consumed by the queued message are accounted on the receiver, but its consumption was obviously triggered by the sender. We must make sure that one peer cannot exhaust the resources of another by queuing messages on it. We do this by applying a per-UID quota logic which works by limiting the number of resources one UID can consume from another to `m/(n+1)`, where `m` is the amount of resources not currently consumed by any other UID, and `n` is the number of UIDs currently consuming any resources of the UID in question. This guarantees that each UID will always be able to consume at least `1/n^2` of the total number of resources of another, where `n` is the number of peers on the bus, and in the common case it will actually get `1/(n+1)`, where `n` is the number of peers currently actively consuming resources of the peer in question, which is pretty close to a perfect division (which would be `1/n`).