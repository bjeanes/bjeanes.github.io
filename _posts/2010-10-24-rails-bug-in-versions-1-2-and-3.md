---
title: "Rails bug in versions 1, 2, and 3"
layout: 'post'
author: bo
---

Want to know how to raise a 500 error in any version of rails (so far)?
Simply add `?a&a[]` to the end of any URL in any Rails application.

I tested this on a number of known rails sites such as Chargify,
Twitter, Groupon, Hoptoad (oh the irony), and even a Rails v1 website
somebody managed to find.

For such an old bug (essentially as old as it can be), it’s quite funny
that it was apparently [patched less than two weeks
ago](https://rails.lighthouseapp.com/projects/8994/tickets/3030-rails-does-not-handle-typeerror-from-rack-on-malformed-query-strings).

