* Log on client forced-disconnect, include metadata in journal, and consider documenting each scenario in detail
* Implement SO_PEERSEC upstream and use to get driver credentials (see https://www.spinics.net/lists/selinux/msg22675.html)
* Implement `send_broadcast`, `max_unix_fds`, `min_unix_fds` in policy language
* AppArmor support
* Rewrite MATCH API to be more convenient to use
* Make NSS-lookups *opt-in*, so we will not use them by default (only the passwd-cache).
* update driver to new dbus-spec extensions
* run as uid specified in xml-config