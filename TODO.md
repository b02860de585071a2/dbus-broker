* Make use of SO_PEERSEC for socketpair(2) controller sockets
* Rewrite MATCH API to be more convenient to use
* update driver to new dbus-spec extensions
* document journal KEYS that we use
* strip unknown header fields
* log quota messages once per peer, not once per user (see 31ba421ef969)
* log startup/shutdown summary in launcher and broker
* driver_parse_metadata() should be implicit for messages we create