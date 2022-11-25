+++
aliases = ["2009/ultraviolet-tools", "2009/07/ultraviolet-tools"]
date = 2009-07-30T04:49:00Z
slug = "ultraviolet-tools"
title = "Ultraviolet Tools"
updated = 2011-08-28T21:31:38Z
+++

A few weeks ago, I was modifying my clone of Enki (my blogging platform)
to use [Ultraviolet](http://ultraviolet.rubyforge.org/) instead of
[CodeRay](http://coderay.rubychan.de/), as it supports using both
[TextMate](http://macromates.com) themes and syntaxes. This means I can
extend it fairly limitlessly and use TextMate itself to modify the
themes an syntaxes.

However, Ultraviolet doesn’t come with the best tools to turn the
TextMate syntax and theme files into files Ultraviolet can use, and the
ones that it does provide puts those translated files INTO the gem on
your system. This means that you can’t easily deploy or share them.

So, to remedy this I made a small gem [Ultraviolet
Tools](http://github.com/bjeanes/ultraviolet-tools) that provides two
command line tools: `uv-create-syntax` and `uv-create-theme`.

They aren’t very complicated, nor very documented, but they are
straight-forward to use:

### Install

Install Ultraviolet Tools with
`sudo gem install bjeanes-ultraviolet-tools -s http://gems.github.com`.

### Usage

#### Themes

Run `uv-create-theme path/to/theme.tmTheme`

#### Syntaxes

Run `uv-create_syntax path/to/syntax.plist` or
`uv-create_syntax path/to/bundle/with/syntaxes.tmBundle`

### Using the resulting files

Use [my fork of ultraviolet](http://github.com/bjeanes/ultraviolet) as
it has an option to change the load path for themes and syntaxes so that
you can load them out of your application files.
