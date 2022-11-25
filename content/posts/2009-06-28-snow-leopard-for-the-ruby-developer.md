+++
aliases = ["2009/snow-leopard-for-the-ruby-developer", "2009/06/snow-leopard-for-the-ruby-developer"]
date = 2009-06-28T02:07:00Z
slug = "snow-leopard-for-the-ruby-developer"
title = "Snow Leopard for the Ruby Developer"
updated = 2011-08-28T21:25:48Z
+++

After the 2009 WWDC release, I decided to give Snow Leopard a try on my
production machine. Crossing my fingers, expecting a plethora of broken
gems and databases, I wiped my hard drive clean and installed the latest
build.

Much to my surprise, most things worked flawlessly and exactly as it did
on Leopard.

### What Ships with Snow Leopard

```bash
$ gem --version
1.3.1

$ ruby --version
ruby 1.8.7 (2008-08-11 patchlevel 72) [universal-darwin10.0]

$ gem list

*** LOCAL GEMS ***

actionmailer (2.2.2, 1.3.6)
actionpack (2.2.2, 1.13.6)
actionwebservice (1.2.6)
activerecord (2.2.2, 1.15.6)
activeresource (2.2.2)
activesupport (2.2.2, 1.4.4)
acts_as_ferret (0.4.3)
capistrano (2.5.2)
cgi_multipart_eof_fix (2.5.0)
daemons (1.0.10)
dnssd (0.6.0)
fastthread (1.0.1)
fcgi (0.8.7)
ferret (0.11.6)
gem_plugin (0.2.3)
highline (1.5.0)
hpricot (0.6.164)
libxml-ruby (1.1.2)
mongrel (1.1.5)
needle (1.3.0)
net-scp (1.0.1)
net-sftp (2.0.1, 1.1.1)
net-ssh (2.0.4, 1.1.4)
net-ssh-gateway (1.0.0)
rails (2.2.2, 1.2.6)
rake (0.8.3)
RedCloth (4.1.1)
ruby-openid (2.1.2)
ruby-yadis (0.3.4)
rubynode (0.1.5)
sqlite3-ruby (1.2.4)
termios (0.9.4)
xmpp4r (0.4)
```

Quite possibly these default gems and versions will change between now
and the final release of Snow Leopard. For instance, this seed comes
with Rails 2.2.2, even though we are already at 2.3.2. Snow Leopard
doesn’t come out till September and by then we could have an even newer
version of Rails that might be bundled.

### What doesn’t work, and how to fix it.

**Textmate** - Everything works fine except for the following shortcuts:
⌘←, ⌘→, ⌘⇧←, and ⌘⇧→. These shortcuts are for moving the text insertion
point to beginning and end of line, and selecting to beginning and end
of line, respectively. The fix is easy, simply download and double click
the TextMate macros attached to [this
ticket](http://ticket.macromates.com/show?ticket_id=0FDE7076)

**Nokogiri** - Nokogiri installs fine but gives error “native.bundle:
mach-o, but wrong architecture” when used. Getting around this is as
easy as running
`sudo gem install nokogiri -s http://tenderlovemaking.com/` to get the
latest development release which includes a fix for this.

**do_sqlite3** - I haven’t personally used this or tried to fix it but
I saw report of it not working [by
@benlovell](http://twitter.com/benlovell/status/2182275909)

**Passenger PrefPane** - Preference pane does not load. No known fix
(it’s probably an easy fix but I haven’t looked enough into it to try).
