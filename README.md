shell_command_lock - Atomic locking for shell commands.

Prevent identical command lines from executing concurrently.
A command line is the combination of the command and it's arguments: $0 $*

PUBLIC DOMAIN

https://github.com/jakeogh/shell_command_lock

Requires: sh, sha1sum

Steps:

1. Generate unique and reproducible string from $0 $* that can be represented as a file name. sha1($0 $*) is used.

2. Obtain atomic lock

3. Write $$ to the lockfile

More info:

 - man flock
 - http://www.davidpashley.com/articles/writing-robust-shell-scripts.html
 - http://wiki.bash-hackers.org/howto/mutex
 - http://mywiki.wooledge.org/BashFAQ/045
 - http://wiki.grzegorz.wierzowiecki.pl/code:mutex-in-bash
 - http://code.google.com/p/pylockfile/
 - https://github.com/skx/sysadmin-util/blob/master/with-lock
 - https://github.com/jaysoffian/dotlock
 - http://sysadvent.blogspot.com/2008/12/day-9-lock-file-practices.html
 - http://apenwarr.ca/log/?m=201012#13
 - https://news.ycombinator.com/item?id=2000349

Notes:

This script attempts to strictly POSIX (no extensions) compliant.

It does not depend on bash specific features.

Redirection using noclobber is the atomic locking primitive used instead of mkdir because in it's faster.

Benchmarks:
``` sh
$ time for x in {1..24000} ; do /bin/mkdir lock ; /bin/rmdir lock ; done
```
``` sh
$ time for x in {1..24000} ; do set -o noclobber; :> lock ; /usr/bin/unlink lock ; done
```

Install: Place in $PATH.

```sh
$ mkdir ~/bin ; mv shell_command_lock ~/bin/shell_command_lock
```

To use, insert:
```
source shell_command_lock
```
or
```
. shell_command_lock #(avoids the 'source' bashism)
```
before the critical section in the parent script. The lock is removed when
the parent script terminates via the trap below.

This script should have no effect on the parent script other than locking. The variable names are set to readonly to prevent silent collisions with names in the parent script. The set commands are done in subshells so we don't need to save and restore the state.

Known Bugs:

- 1. IMPORTANT: If the trap is re-defined in the parent script, then that trap will need to handle deleting the lock.
- 2. The lockfile is orphaned if a exit signal happens after the lock is obtained and before trap is set.
