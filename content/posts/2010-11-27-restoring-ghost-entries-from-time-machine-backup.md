+++
aliases = ["2010/restoring-ghost-entries-from-time-machine-backup", "2010/11/restoring-ghost-entries-from-time-machine-backup"]
date = 2010-11-27T05:35:00Z
slug = "restoring-ghost-entries-from-time-machine-backup"
title = "Restoring ghost entries from Time Machine backup"
updated = 2011-08-28T19:09:23Z
+++

Pat Allan just had some trouble with his laptop and he lost a bunch of
data.

After he pinged me asking where ghost stored it’s hostname entries so he
could restore them, I realised I didn’t know. Sure, on Linux, ghost just
writes out to the `/etc/hosts` file, but on OS X, it uses the `dscl`
command to get to the Directory Services.

Long story short, if you ever want to know where and how Directory
Services stores a lot of it’s information, it’s mostly in the
`/var/db/dslocal/nodes/Default/` directory.

Specifically, the hosts are stored in
`/var/db/dslocal/nodes/Default/hosts/` directory, one `.plist` file per
host. Pat reports that just grabbing those files from a his backup and
dropping them in that directory worked wonders.