DBus is meant to be a reliable transport, this poses several challenges, this page outlines how the dbus-broker deals with them.

## Fatal errors

All internal errors in the broker are considered fatal, all peers are forcibly disconnected
and the broker is shut down. We do not try to work around issues due to say memory shortage or
configuration errors.

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

The amount of resources for message queuing is limited, and in case the necessary resources
for queueing a message is not available, this is dealt with as follows:

### Method calls

In case NO_REPLY_EXPECTED is specified the message is silently dropped, otherwise an error
message is returned from the driver.

### Method returns and errors

We guaranteed that all replies are delivered, so if this cannot be done due to lack of
resources the recipient is shutdown. This means that no more messages are queued on the
recipient and once all the pending messages have been flushed to the kernel the socket is
shutdown. This guarantees that messages that have already been queued are not affected by
the resource exhasution, but that after a message is dropped the peer is disconnected and
no further messages can be received. It is still up to the remote end to call close() to
trigger the actual disconnect from the bus.

### Expected signals

Like method returns, expected signals, i.e., signals the receiver subscribed to, are guaranteed
to be delievered, and if that fails due the situation is handled like replies.

### Unexpected signals

It is possible to send a directed signal to a peer without the peer having subscribed to it.
This is treated similarly to a method call with NO_REPLY_EXPECTED, so if the message cannot
be delivered it is silently dropped. This behavior is discouraged, but kept for backwards
compatibility.