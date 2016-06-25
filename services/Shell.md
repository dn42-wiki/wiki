# Shell

Providers:

| Person        | Hostname                             | Net              | Description | Contact       |
|:------------- |:------------------------------------ |:---------------- |:----------- |:------------- |
| aix           | entropy.aix.ovh & entropy.aix.dn42   | clearnet & dn42  | See below   | aix @ hackint |
| mortzu        | shell.mortzu.dn42                    | dn42 only        | -           | -             |

## Entropy shellbox
The Entropy shellbox runs a [Grsecurity](https://grsecurity.net/) secured kernel, along with various other hardening features such as [RBAC](https://en.wikipedia.org/wiki/Role-based_access_control) and some sysctl tweaks. It has an internal mail system which anyone can use to contact a shell user ([user]@entropy.aix.ovh). Additionally, it has all of the [BlackArch tools](http://www.blackarch.org/tools.html) installed and available for everyone to use.
To further enhance security, the ownership of various SUID executables and logs has been restricted to members of certain groups, which are nicely explained by the [MOTD](https://entropy.aix.ovh/shell/motd):
```
        tpe: allows you to execute files not in root-owned
                directories writeable only by root

        nosock: cannot open any sockets
        noclisock: cannot open client sockets
        noservsock: cannot open server sockets

        viewproc: can see all processes on the system

        suid: can run `su`, `sudo`, `gpasswd` and `chage`
        usrsuid: can run `newgrp`, `chsh`, `chfn` and `at`

        share: can write to `/srv/share`

        snoop: can see users logged on to the system
                and their addresses

        msg: can run `wall` and `write`

        cron: can use the cron system

        volumes: can use `mount`, `umount` and `mount.nfs`

        audit: your activities are logged
```

By default, users will be part of only `tpe` and `audit` groups (as well as their own) but may request to be added to other groups. Please note that the only execve() and chdir() calls are logged as a result of a user's membership of the `audit` group.

Lastly, in the interests of full disclosure, here are the [details of the box](http://pastie.org/pastes/10889893/text)(Subject to change).