+++
aliases = ["2009/using-fish-shells-event-system-to-behave-like-method-missing", "2009/10/using-fish-shells-event-system-to-behave-like-method-missing"]
date = 2009-10-07T09:07:00Z
slug = "using-fish-shells-event-system-to-behave-like-method-missing"
title = "Using Fish shell's event system to behave like method_missing"
updated = 2011-08-28T21:03:23Z

[taxonomies]
tags = ["shell"]
+++

Dr Nic’s latest post, [hash bang
cucumber](https://web.archive.org/web/20170911072928/http://drnicwilliams.com/2009/10/07/hash-bang-cucumber/), reminded
me of a piece of hax I whipped up a few weeks ago with Ruby and Fish Shell.

Fish has an event system that allows you to register functions to be auto-run after or during certain events (such as
when a particular environment variable is changed). One of this events is called `fish_command_not_found`. It is
triggered whenever you type a non-existant command into the prompt.

With this in mind, you can trigger certain commands to run by matching what fish couldn’t manage to run automatically by
catching this event.

For instance:

``` fish
function __fish_method_missing --on-event fish_command_not_found
  method_missing $argv
end

funcsave method_missing
```

Given that function is created (simply paste the whole code block into your Fish terminal), I can now create a command
called `method_missing` (or whatever you call inside your `__fish_method_missing`) and place it somewhere in your
`$PATH` — I like `~/.config/fish/bin`.

My `method_missing` binary is simply something like this:

``` ruby
#!/usr/bin/env ruby

command = ARGV.shift

def run(cmd)
  puts "Running #{cmd.inspect} instead"
  system(cmd)
end

case command
when /^git(@|:\/\/).*\.git$/
  run("git clone #{command.inspect}")
when /^(?:ftp|https?):\/\/.+\.t(?:ar\.)?gz$/
  run("curl #{command.inspect} | tar xzv")
else
  $stderr.puts "No default action defined in #{__FILE__.inspect}"
  abort
end
```

Each command you want to register just becomes a new `when`
statement. For instance, to implement the functionality Dr Nic was
trying to achieve, I simply modify the `case` block as such:

``` ruby
case command
when /^[a-z0-9_\-\/]+\.feature(:\d+)?$/
  run("cucumber #{command}")
# ...
end
```

The existing entries I have in my `method_missing` command will auto-clone the repository of a pasted Git URL and
download and expand a URL for a tar file, respectively. Not too shabby, and dead easy to implement.
