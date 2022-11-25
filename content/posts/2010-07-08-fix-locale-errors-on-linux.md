+++
aliases = ["2010/fix-locale-errors-on-linux", "2010/07/fix-locale-errors-on-linux"]
date = 2010-07-08T08:50:00Z
slug = "fix-locale-errors-on-linux"
title = "Fix locale errors on linux"
updated = 2011-08-28T20:50:23Z
+++

Have you ever gotten error messages such as the following when running
commands such as `apt-get`?

``` text
perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
 LANGUAGE = ,
 LC*ALL = ,
 LC*CTYPE = "en*US.UTF~~8",
 LANG = 
 are supported and installed on your system.
```

I’ve had this problem quite a lot on my VPS boxes on Linode and
SliceHost. The solution is simple, simply run the following:

``` console
locale-gen en_US.UTF-8
update-locale LANG=en_US.UTF-8
```
