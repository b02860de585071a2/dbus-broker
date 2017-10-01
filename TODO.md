* Implement ReloadConfig
* Make SIGHUP trigger ReloadConfig and hook up in service file `ExecReload`
* Add `c_list_for_each_continue()` and use with match-iterators
* Make `c_rbtree_remove()` not require the root-pointer
* Test-infrastructure should check `-O3`
* Implement destination-matches for monitors (see also https://bugs.freedesktop.org/attachment.cgi?id=119441)
* Implement `send_broadcast`, `max_unix_fds`, `min_unix_fds` in policy language