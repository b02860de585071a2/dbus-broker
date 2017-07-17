This page describes how to integrate dbus-broker without a compatibility shim. This is useful if what one wants is the message broker, but without configuration-file compatibility with the reference implementation. The main benefit is that dbus-broker behaves like an (out-of-process) library, without any knowledge of (or access to) its environment (no file-system access, no IPC of its own, no privileges, etc). As such is it perfectly suited for private buses or to give programmatic control of the broker to a parent/owning process.

## Spawning

We refer to the dbus-broker binary as `the broker` and the parent process as `the controller`. The controller forks off the broker, and passes in one end of a UDS socket pair to the broker, which is a peer-to-peer D-Bus connection between the controller and the broker. This connection (and what is passed in over it) is the only communication channel out of the broker.

## Listening

In order for clients to connect to the broker, the controller must establish a listening socket the clients can connect to, and pass this to the broker (by performing a method call over the private D-Bus connection) for it to listen to. Together with the listening socket, the controller must pass in a policy database that will be applied to any clients connecting to the socket.

In order to create a broker compatible with the reference implementation, the listening socket should be installed by the controller in the standard filesystem location, and the passed-in policy should be generated from the XML policy found in the default location in the filesystem (both these locations depend on whether there is a system or a user bus that is being implemented).

## Names

In order for names to be activatable, the controller must call a method on the private broker connection to enable each name for activation. It will also pass in which uid the resources of the name (which may otherwise be unbounded) should be accounted on. Ideally this uid should be the same as the one eventually owning the name (if this is known by the controller).

Again, if the broker is meant to be a drop-in replacement for the reference implementation, the activatable names should be read from the standard file-system locations and installed into the broker before the listening socket is (so they are static and always available).

In general though, names may be made activateable (and removed again) at any point in time by the controller.

## Activation

When a peer requests activation (explicitly or implicitly) of an activatable name, a signal is sent to the controller over the private socket indicating the request. However, it is the sole responsibility of the controller to know how to handle such a request (the broker does not know anything about the implementers of names). Ideally, the controller is also the process responsible for spawning clients (it could for instance be PID1 for the system bus), but it could also simply forward the requests to (the equivalent of) PID1.

Requests made by peers to the broker to update the activation environment, are simply forwarded to the controller, which is responsible for making sure any further peers are spawn with the correct environment variables set.