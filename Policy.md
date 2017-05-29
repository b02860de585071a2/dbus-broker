This defines the policy implemented by the dbus broker.

# Users

By default, any user may connect to the broker. Optionally, a blacklist or a whitelist (but not both) of uid's and one of gid's may be provided to only allow some peers to connect. The list of uid's take precedent over the list of gid's.

# Names

By default, any peer may own any name. However, a blacklist or a whitelist (but not both) may be provided to only allow some names to be owned. Moreover, such a list may also be provided once per gid and once per uid. The per-gid list takes precedence over the global list and the per-uid list takes precedence over the per-gid one.

# Message transactions

By default, any peer may send a method call or a directed signal to any other, and any peer may reply with a method reply or an error to any method call it has received, and any peer may broadcast any message to anyone subscribed to it and any peer may eavesdrop on any other.

However, restrictions can be made using transaction policies. A policy comes in four varieties: a send policy, a receive policy, and for each a eavesdropping and a non-eavesdropping variety. Transaction policies are similar to name policies in that there is one global one, one per gid, one per uid, and the most specific one applies to a given peer. A given message may be transmitted if it is both allowed to be sent and to be received according to the corresponding policies.

A policy consists of a set of rules mapping the tuple (name, path, interface, member, type) to either 'allow' or 'deny'. Each member of the tuple may be a wildcard. For a message to be allowed by a given policy, each tuple formed by combining each of the names of the sending/receiving peer as well as the wildcard name with the values from the message's metadata must map to 'allow'.

More specific rules take precedence over less specific ones. A rule is considered more specific than another if they both match a given message, and the former has a literal and the latter a wildcard in the first member field of the tuple in which they differ.

# dbus-daemon compatibility

The policy files of the dbus-daemon may be compiled into the policies accepted by the broker with the following caveats:
 - 'at_console=true' rules must be discarded,
 - it is never possible to have a policy to cause non-expected replies to be delivered.