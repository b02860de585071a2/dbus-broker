* Log on client forced-disconnect, include metadata in journal, and consider documenting each scenario in detail
* Implement SO_PEERSEC upstream and use to get driver credentials (see https://www.spinics.net/lists/selinux/msg22675.html)
* Implement `send_broadcast`, `max_unix_fds`, `min_unix_fds` in policy language
* AppArmor support
* Add `c_list_for_each_continue()` and use with match-iterators
* Rewrite MATCH API to be more convenient to use