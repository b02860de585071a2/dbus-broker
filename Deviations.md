While dbus-broker aims to keep perfect compatibility to existing D-Bus implementations, the D-Bus specification, as well as the reference implementation, there are several occasions where it deviates. Usually, the reason behind those deviations is to mitigate possible attacks, or to improve performance. The complete list of known deviations is as follows:

* **Reply Policies**: While the reference-implementation supports applying D-Bus policies to method-returns, as well as filtering whether a reply is expected or not, dbus-broker does not support this. Moreover, dbus-broker only allows expected replies, and those are allowed unconditionally.
_Unexpected-replies_ and _Reply-filtering_ have no known users (nor use-cases), hence support has been dropped.

* **at_console Policies**: D-Bus policies in the `at_console=true` context are ignored by dbus-broker. Moreover, the broker considers all peers `at_console=false`, hence only applies those rules. Inspecting whether a peer is considered `at_console` requires IPC at runtime. However, we strongly believe that an implementation of IPC should never perform recursive IPC at runtime to provide its service. Furthermore, `at_console` rules are deprecated by the reference implementation, and there are suitable replacements like PolKit available.

* **WIP**