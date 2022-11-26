+++
aliases = ["2009/fix-for-dynamic-pager-cannot-open-swap-directory-private-var-vm-in-os-x-possibly-just-snow-leopard", "2009/07/fix-for-dynamic-pager-cannot-open-swap-directory-private-var-vm-in-os-x-possibly-just-snow-leopard"]
date = 2009-07-09T05:34:00Z
slug = "fix-for-dynamic-pager-cannot-open-swap-directory-private-var-vm-in-os-x-possibly-just-snow-leopard"
title = "Fix for \"dynamic_pager: cannot open swap directory /private/var/vm\" in OS X (possibly just Snow Leopard)"
updated = 2011-08-28T21:07:39Z

[taxonomies]
tags = ["mac"]
+++

Today my Snow Leopard install was going so slow it felt like I was trying to swim through molasses. The system was
literally so slow that typing text into some text fields (especially TextMate) lagged at about a rate of 1 character per
second, and opening files (or even running `touch` on them) took a couple of seconds.

After a few fruitless restarts and a keen eye on my logs I started noticing these errors in Console.app:

```
9/07/09 3:01:37 PM com.apple.launchd[1] (com.apple.dynamic*pager) Exited with exit code: 1
9/07/09 3:01:37 PM com.apple.launchd Throttling respawn: Will start in 10 seconds
9/07/09 3:01:47 PM com.apple.dynamic_pager[2481] dynamic_pager: cannot open swap directory /private/var/vm
9/07/09 3:01:47 PM com.apple.launchd Exited with exit code: 1
9/07/09 3:01:47 PM com.apple.launchd Throttling respawn: Will start in 10 seconds
9/07/09 3:01:57 PM com.apple.dynamic_pager[2546] dynamic_pager: cannot open swap directory /private/var/vm
9/07/09 3:01:57 PM com.apple.launchd Exited with exit code: 1
9/07/09 3:01:57 PM com.apple.launchd Throttling respawn: Will start in 10 seconds
9/07/09 3:02:07 PM com.apple.dynamic_pager[2547] dynamic_pager: cannot open swap directory /private/var/vm
9/07/09 3:02:07 PM com.apple.launchd[1] (com.apple.dynamic_pager[2547]) Exited with exit code: 1
9/07/09 3:02:07 PM com.apple.launchd[1] (com.apple.dynamic_pager) Throttling respawn: Will start in 10 seconds
```

I don’t know how or why they started but I was able to fix it by simply running `sudo mkdir /private/var/vm` in
Terminal. Within a few seconds of creating the directory, the errors ceased and I could see a swapfile created inside
the directory.

I have no idea what caused the directory to be deleted in the first place or why OS X isn’t smart enough to try creating
it, but the fix is simple and instantly effective.

Note: I am not sure if this is a Snow Leopard specific issue or not, but it very well might be. Also, these started
right after installing Adobe CS4, so I feel pretty confident blaming that for now.
