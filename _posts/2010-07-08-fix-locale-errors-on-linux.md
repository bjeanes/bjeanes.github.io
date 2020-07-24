---
title: "Fix locale errors on linux"
layout: 'post'
author: bo
---

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

