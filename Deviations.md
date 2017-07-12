While dbus-broker aims to keep perfect compatibility to existing D-Bus implementations, the D-Bus specification, as well as the reference implementation, there are several occasions where it deviates. Usually, the reason behind those deviations is to mitigate possible attacks, or to improve performance. The complete list of known deviations is as follows:

* **Reply Policies**: While the reference-implementation supports applying D-Bus policies to method-returns, as well as filtering whether a reply is expected or not, dbus-broker does not support this. Moreover, dbus-broker only allows expected replies, and those are allowed unconditionally.
_Unexpected-replies_ and _Reply-filtering_ have no known users (nor use-cases), hence support has been dropped.

* **at_console Policies**: D-Bus policies in the `at_console=true` context are ignored by dbus-broker. Moreover, the broker considers all peers `at_console=false`, hence only applies those rules. Inspecting whether a peer is considered `at_console` requires IPC at runtime. However, we strongly believe that an implementation of IPC should never perform recursive IPC at runtime to provide its service. Furthermore, `at_console` rules are deprecated by the reference implementation, and there are suitable replacements like PolKit available.

* **Recursive FD Passing**: The linux kernel allows queuing a socket on itself. If a client passes a message with its own end of the Unix-Domain-Socket as payload, the message will keep the client alive, even if the original client exits and closes its FDs. As such, if the message is directed at itself, there will be no way of disconnecting that client.
While dbus-daemon applies timeouts to pending messages (and thus disconnects some of those clients after 120s), dbus-broker does not cleanup such clients. Instead, dbus-broker relies on its resource accounting and ignores such clients. Hence, dbus-broker does not distinguish such clients from live clients.

* **Direct Activation**: Historically, all activated services were forked and exec'ed by the respective bus implementation. With the introduction of systemd, activated service could opt-in to be run in a systemd service instead. With dbus-broker, all activated services are run as systemd service (possibly as TransientUnit if none is provided by the service).

* **WIP**