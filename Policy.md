The D-Bus specification specifies three kinds of policies. This page outlines how
the dbus broker differs in its implementation of the policies compared to the specification
and the reference implementation.

== Credentials ==

Whenever a peer connectns to the bus, a policy is generated for the peer based on its
credentials. In order to make this intuitive and predictable we base the decision
entirely on the uid, gid and auxillary groups pinned in the socket by the calling
peer at connect(). In particular, we implicitly assume at_console=false. Also, we
do not do any resolution of groups over nss at connect time, unlike the way the
reference implementation works. This avoids a race when groups could change after
connect and more importantly avoids a potential dead-locke due to non-trivial nss
modules. This is made possible by a new kernel feature (SO_PEERGROUPS), which the
reference implementation could also make use of, if they so choose.

== Replies ==

The specification allows unexpected replies to be allowed to be transmitted, or
expected ones to be refused. We do not allow any policy on replies, rather we
always transmit expected replies and never unexpected ones. This feature does
not appear to be used, and supporting it would make our code more fragile and
seems to open up for more problems than it solves.

== Policy Language ==

Appart from the above, we are fully compatible with the policy language of the
reference implementation, though we express it differently in our API. The
language we support is closer to how it is actually used, though not necessarily
how it is typically expressed in configuration files. There are four types of,
connect, own, send and receive. The former determines which peers are allowed
to connect to the bus, and the latter three are attached to each peer at connect
time and determines their behavior.

=== Policy evaluation ===

Whenever deciding whether a certain action is permitted by a policy or not, several
queries may be made to the policy database which should all be combined to form the
final decision. The logic here is always the same, a judgement always indicates if
something is 'allowed' or 'denied', and it indicates the priority of the judgement.
If several judegements are made, the one with the higher priority takes precedent.

=== Connect ===

The connect policy is given by a wildcard judgement, and up to one judgement per
uid and gid. Whenever a peer connects to the daemon a wildcaerd query is made and
one query for the peers uid as well as one for each of its auxillary gids. As
explained above, the judgement with the higher priority takes precedent.