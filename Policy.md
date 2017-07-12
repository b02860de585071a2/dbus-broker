The D-Bus specification specifies three kinds of policies. This page outlines how
dbus-broker differs in its implementation of the policies compared to the specification
and the reference implementation.

## Credentials

Whenever a peer connects to the bus, a policy is generated for the peer based on its
credentials. In order to make this intuitive and predictable we base the decision
entirely on the uid, gid and auxiliary groups pinned in the socket by the calling
peer at `connect()`. In particular, we implicitly assume `at_console=false`. Also, we
do not do any resolution of groups over `nss` at connect time, unlike the way the
reference implementation works. This avoids a race when groups could change after
connect and more importantly avoids a potential dead-lock due to non-trivial `nss`
modules. This is made possible by a new kernel feature (SO_PEERGROUPS), and as a
transitional measure, we will still use `nss` (and warn) until that feature is
available.

## Replies

The specification allows unexpected replies to be transmitted and expected ones
to be refused. We do not allow any policy on replies, rather we always transmit
expected replies and never unexpected ones. This feature does not appear to be
used, and supporting it would make our code more fragile and seems to open up
for more problems than it solves.

## Policy Language

Apart from the above, we are fully compatible with the policy language of the
reference implementation, though we express it differently in our API. The
language we support is closer to how it is actually used, though not necessarily
how it is typically expressed in configuration files. There are four types of
policies: connect, own, send and receive. The former determines which peers are
allowed to connect to the bus, and the latter three are attached to each peer at
`connect()` time and determines their behavior. The latter three policy types all
have a wildcard policy, a per-uid policy and a per-gid policy. At connect, the uid
policy of the connecting peer (or the wildcard one if none exists) as well as the
gid policy for each of the connecting peer's auxiliary groups are attached to the
peer to form its policy.

### Policy evaluation

Whenever deciding whether a certain action is permitted by a policy or not, several
queries may be made to the policy database which should all be combined to form the
final decision. The logic here is always the same, a judgement always indicates if
something is 'allowed' or 'denied', and it indicates the priority of the judgement.
If several judgments are made, the one with the higher priority takes precedent.
The default evaluation, in case there are no rules installed is 'deny'.

### Connect

The connect policy is given by a wildcard judgement, and up to one judgement per
uid and gid. Whenever a peer connects to the daemon a wildcard query is made and
one query for the peers uid as well as one for each of its auxiliary gids. As
explained above, the judgement with the higher priority takes precedent.

### Own

An own policy is given by a judgement for each of a set of names, and for a set of name
prefixes (including the wildcard prefix). When a peer requests ownership of a name a
query is made to the peer's own policy of the name itself and each of its prefixes, the
returned judgments are combined in the usual way.

### Send

A send policy consists of one by-name policy for each of a set of well-known names, as well as a
wildcard by-name policy. Each of the by-name policies assign judgments to message metadata, as
is done by the regular dbus policies (matching on type, object path, interface and member).

When a message is sent, queries are made to the sender's send policies. The by-name policies
corresponding to each of the well-known names owned by the receiver are queried with the message
metadata as input, as well as the wildcard by-name policy. The returned judgments are combined in
the standard way.

In case sending and receiving is not atomic, the policy of the sender is the one at send, and the
well-known names of the receiver are those at receive.

### Receive

A receive policy is exactly as a send policy, though it is applied in the inverse way: The receive
policy of the receiver at receive is queried using the well-known names of the sender at send.