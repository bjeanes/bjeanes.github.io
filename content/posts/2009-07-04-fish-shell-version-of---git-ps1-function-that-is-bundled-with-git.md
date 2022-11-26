+++
aliases = ["2009/fish-shell-version-of---git-ps1-function-that-is-bundled-with-git", "2009/07/fish-shell-version-of---git-ps1-function-that-is-bundled-with-git"]
date = 2009-07-04T15:34:00Z
slug = "fish-shell-version-of---git-ps1-function-that-is-bundled-with-git"
title = "Fish Shell version of __git_ps1 Function That is Bundled with Git"
updated = 2011-08-28T21:04:52Z

[taxonomies]
tags = ["shell"]
+++

Today at ActionHack, I was showing off Fish Shell to [@geoffreyd](http://twitter.com/geoffreyd). I was showing off all
the cool fish prompt features I had, but pointed out that my git branch portion needed some beefing up, as it didn't
show the current mode nor commit shas when not in a branch.

Knowing that git came with a `__git_ps1()` function for Bash that achieves this, I decided to port it to Fish tonight.
My shell scripting fu is pretty weak but as far as I can tell it works great and I am now using it in my shell.

Here is the Fish function (I’ve kept the function name the same as the bash one):

``` bash
function __git_ps1
  set -l g (git rev-parse --git-dir ^/dev/null)
  if [ -n "$g" ]
    set -l r ""
    set -l b ""

    if [ -d "$g/../.dotest" ]
      if [ -f "$g/../.dotest/rebasing" ]
        set r "|REBASE"
      elseif [ -f "$g/../.dotest/applying" ]
        set r "|AM"
      else
        set r "|AM/REBASE"
      end

      set b (git symbolic-ref HEAD ^/dev/null)
    elseif [ -f "$g/.dotest-merge/interactive" ]
      set r "|REBASE-i"
      set b (cat "$g/.dotest-merge/head-name")
    elseif [ -d "$g/.dotest-merge" ]
      set r "|REBASE-m"
      set b (cat "$g/.dotest-merge/head-name")
    elseif [ -f "$g/MERGE_HEAD" ]
      set r "|MERGING"
      set b (git symbolic-ref HEAD ^/dev/null)
    else
      if [ -f "$g/BISECT_LOG" ]
        set r "|BISECTING"
      end

      set b (git symbolic-ref HEAD ^/dev/null)
      if [ -z $b ]
        set b (git describe --exact-match HEAD ^/dev/null)
        if [ -z $b ]
          set b (cut -c1-7 "$g/HEAD")
          set b "$b..."
        end
      end
    end

    if not test $argv
  		set argv " (%s)"
  	end

  	set b (echo $b | sed -e 's|^refs/heads/||')

    printf $argv "$b$r" ^/dev/null
  end
end
```

Enjoy :)
