---
title: "Unescaping UTF-8 Strings in Ruby 1.9"
layout: 'post'
author: bo
---

Today [Radar](http://frozenplague.net) and I encountered a little issue
with `String` encodings in Ruby 1.9. In this project, some Merb UTF-8
params needed to be unescaped. We spent some trying to force strings
into UTF-8 encoding but for some reason while the encoding was UTF–8,
the actual contents of the strings were getting massacred.

Long story short:

``` ruby
# In Ruby 1.9

# Broken
puts URI::unescape("Baden-W%C3%BCrttemberg") # => "Baden-WÃ¼rttemberg"

# Working
puts CGI::unescape("Baden-W%C3%BCrttemberg") # => "Baden-Württemberg"
```

Another lesson we learnt on our adventures today is the following:

``` ruby
m1 = "Munich"
puts = m1.encoding  # let's suppose it is something other than UTF-8, such as ASCII-8Bit

m2 = "München"
puts = m2.encoding  # let's suppose we have a UTF-8 encoded string


# The following will raise an exception about mismatched encodings
m3 = m1 << m2

# The easiest way to get around this is to do the concatenation like so:
m3 = m3 = m1 << m2.force_encoding("UTF-8")
```
