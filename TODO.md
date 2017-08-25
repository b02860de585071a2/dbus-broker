* Implement ReloadConfig
* Make SIGHUP trigger ReloadConfig and hook up in service file `ExecReload`
* Add `c_list_for_each_continue()` and use with match-iterators
* Make `c_rbtree_remove()` not require the root-pointer