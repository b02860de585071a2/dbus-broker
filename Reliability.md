DBus is meant to be a reliable transport, this poses several challenges, this page outlines how the dbus-broker deals with them.

## Fatal errors

All internal errors in the broker are considered fatal, all peers are forcibly disconnected
and the broker is shut down. We do not try to work around issues due to say system memory shortage (as opposed to our internal quotas being reached, see below) or configuration errors.

## Protocol violations

If a peer commits a protocol violation it is (possibly following an error reply) gracefully
disconnected. This means that no further messages from the peer are dispatched, all messages
already queued on the peer are still delivered, before the socket is shutdown. We consider
the disconnect as notification to the peer that it did something wrong, even if a proper error
message could not be delivered to it.

## Replies

A method call receives a reply if and only if the NO_REPLY_EXPECTED flag is unset. Either from
the destination peer or from the driver itself in case an error occurred, or the destination
disconnected without ever sending a reply. No other message type ever receives a reply.

## Resource exhaustion 

The amount of resources for message queuing is [limited](Accounting), and in case the necessary resources
for queuing a message is not available, this is dealt with as follows:

### Method calls

In case NO_REPLY_EXPECTED is specified the message is silently dropped, otherwise an error
message is returned from the driver. If, in turn, the error from the driver cannot be delivered, this is handled, as described in the nelike any other error message, as 

### Method returns and errors

We guaranteed that all replies are delivered, so if this cannot be done due to lack of
resources the recipient is shutdown. This means that no more messages are queued on the
recipient and once all the pending messages have been flushed to the kernel the socket is
shutdown. This guarantees that messages that have already been queued are not affected by
the resource exhaustion, but that after a message is dropped the peer is disconnected and
no further messages can be received. It is still up to the remote end to call close() to
trigger the actual disconnect from the bus.

### Expected signals

Like method returns, expected signals, i.e., signals the receiver subscribed to, are guaranteed
to be delivered, and if that fails the failure is handled like replies: the recipient is gracefully disconnected.

### Unexpected signals

It is possible to send a directed signal to a peer without the peer having subscribed to it.
This is treated similarly to a method call with NO_REPLY_EXPECTED, so if the message cannot
be delivered it is silently dropped. It is discouraged to rely on this behavior, but (until further notice) we kept this for backwards compatibility. Unexpected signals are discouraged both because they may be silently dropped, but also because clients can easily be confused by receiving unexpected directed signals and treating them as expected broadcasts (which may prove to be a security issue, depending on the particulars of the peer in question).