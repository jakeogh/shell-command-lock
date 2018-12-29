**commandlock - Atomic locking for commands**

https://github.com/jakeogh/commandlock

Prevent identical commands from executing concurrently.

A command is the combination of the program and it's arguments: $0 $*

Requires: sh, sha1sum

**Theory:**

 1. Generate unique and reproducible string from $0 $* (the script name and all it's arguments). sha1sum($0 $*) is used.
 2. Obtain an atomic* lock using the filesystem. Exit on failure (since the same command is already running).

        _At this point, executing the same command (which also uses this script) should fail._

 4. Delete the lockfile on exit.

* filesystem/kernel dependent

**General Install:**

Place in $PATH

```
    git clone https://github.com/jakeogh/commandlock
    sudo cp commandlock/commandlock /usr/bin/commandlock
    sudo cp commandlock/lock /usr/bin/lock
```

**Gentoo Install:**
```
   su
   layman -o https://raw.githubusercontent.com/jakeogh/jakeogh/master/jakeogh.xml -f -a jakeogh
   echo "source /var/lib/layman/make.conf" >> /etc/portage/make.conf
   layman -S
   emerge commandlock
```

**Use:**

simple example:
```
$ lock wget http://www.wrh.noaa.gov/images/twc/granalyst/kemx_cr_0.jpg
```

deliberately attempting to run the same command at the same time:
```
$ lock curl https://www.wrh.noaa.gov/images/twc/granalyst/kemx_cr_0.jpg -o /dev/null & lock curl https://www.wrh.noaa.gov/images/twc/granalyst/kemx_cr_0.jpg -o /dev/null
[1] 26842
ERROR: [26842] /usr/bin/lock curl https://www.wrh.noaa.gov/images/twc/granalyst/kemx_cr_0.jpg -o /dev/null Locking failed: /dev/shm/commandlock_cb201bdd3d9970730fa58471d8f554db966694d3
ERROR: [26842] /usr/bin/lock curl https://www.wrh.noaa.gov/images/twc/granalyst/kemx_cr_0.jpg -o /dev/null is/was already running. Exiting (1).
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  698k  100  698k    0     0   334k      0  0:00:02  0:00:02 --:--:--  334k
[1]+  Exit 1                  lock curl https://www.wrh.noaa.gov/images/twc/granalyst/kemx_cr_0.jpg -o /dev/null

```


or to use it to only lock a specific section of a script, insert:
```
. /usr/bin/commandlock || exit 1
```
_before_ the critical section in the parent script. The lock is removed via the trap when the parent script terminates.


This script should have no effect on the parent script other than locking. The variable names are set readonly to prevent collisions with names in the parent script. The set commands are done in subshells so we don't need to save and restore state.

**Design notes:**

- Attempts to be strictly POSIX compliant.
- Does not depend on bash specific features.
- Redirection using noclobber is the atomic locking primitive instead of mkdir because it's faster.

**Benchmark mkdir vs noclobber:**
```
$ time for x in {1..24000} ; do /bin/mkdir lock ; /bin/rmdir lock ; done
$ time for x in {1..24000} ; do set -o noclobber; :> lock ; /usr/bin/unlink lock ; done
```

**Bugs: (unavoidable?)**

- If the trap is re-defined in the parent script, that trap will need to handle deleting the lock.
- The lockfile is orphaned if an exit signal happens after the lock is obtained and before the trap is set.

**More info:**

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
 - https://github.com/WoLpH/portalocker
