+++
aliases = ["2009/ghost-manage-hostnames-effortlessly", "2009/06/ghost-manage-hostnames-effortlessly"]
date = 2009-06-28T00:13:00Z
slug = "ghost-manage-hostnames-effortlessly"
title = "Ghost - Manage hostnames effortlessly"
updated = 2011-08-28T20:46:59Z
+++

### Background

Ghost is a gem that provides a command and a Ruby API for managing
hostnames locally, much as people do manually by editing `/etc/hosts`
now.

Sometime last year I was having to add lots of
[Passenger](http://modrails.com/) virtual hosts on my development
machine and felt that having to constantly edit the `/etc/hosts` file
was a bit archaic for my liking and that there must be a nicer way of
accomplishing the same task.

Having seen the [Passenger Preference
Pane](http://www.fngtps.com/2008/09/passenger-preference-pane-v1-1)
added hostnames without requiring the user to do anything special, I had
a peak into its [source](http://github.com/alloy/passengerpane) and
discovered the `dscl` ([man
page](http://developer.apple.com/documentation/Darwin/Reference/ManPages/man1/dscl.1.html))
command that comes with Leopard.

For the hell of it I decided to write my
[Ghost](http://github.com/bjeanes/ghost) gem using this command and use
it to make a nice wrapper that has an easier syntax than `dscl`, and
that is less effort than manually editing my hosts file. Happy with the
result, a few other people requested support for other unix systems and
[Mitchell V Riley](http://newtonsinlaws.com) was kind enough to patch
Ghost to fall back to processing and editing the `/etc/hosts` file when
not on Leopard.

### Usage

```bash
$ ghost add mydevsite.local
  [Adding] mydevsite.local -> 127.0.0.1

$ ghost add staging-server.local 67.207.136.164
  [Adding] staging-server.local -> 67.207.136.164

$ ghost list
Listing 2 host(s):
  mydevsite.local      -> 127.0.0.1
  staging-server.local -> 67.207.136.164

$ ghost delete mydevsite.local
  [Deleting] mydevsite.local

$ ghost list
Listing 1 host(s):
  staging-server.local -> 67.207.136.164

$ ghost modify staging-server.local 64.233.167.99
  [Modifying] staging-server.local -> 64.233.167.99

$ ghost list
Listing 1 host(s):
  staging-server.local -> 64.233.167.99

$ ghost empty
  [Emptying] Done.

$ ghost list
Listing 0 host(s):
```

### Other

Since creating Ghost I’ve used the Ruby library to add an initializer to
my Rails projects that add the domain locally if running under Passenger
in development mode (some configuration is needed to let it do this as
it needs sudo).

Also, I have heard reports that entries in the `/etc/hosts` file are
ignored in Snow Leopard. I haven’t tried because I only ever use Ghost
now, but I do know that Ghost still works well.

### Caveats

It only works on Unix-based systems and the `/etc/hosts` path is
hard-coded. Windows support is entirely possible if somebody wants to
use it. It isn’t something I am likely to do as I haven’t booted Windows
in at least a year. In fact, I don’t think I have even ever installed
Ruby on Windows before…
