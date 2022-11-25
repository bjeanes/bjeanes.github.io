+++
aliases = ["2009/gotcha-with-hash-new-when-providing-a-default-value", "2009/10/gotcha-with-hash-new-when-providing-a-default-value"]
date = 2009-10-01T02:01:00Z
slug = "gotcha-with-hash-new-when-providing-a-default-value"
title = "Gotcha with Hash.new when providing a default value"
updated = 2011-08-28T21:31:26Z
+++

For a project I am working on at the moment, I needed a `Hash`
that returned a different default value, i.e. not `nil`.
Specifically, I needed it to return another `Hash`, and for that
internal `Hash`’s default value to be an `Array`, like so:

``` ruby
hash = # ...
p hash.class                              #=> Hash
p hash[:non_existant_key].class           #=> Hash
p hash[:another][:non_existant_key].class #=> Array
```

Naïvely, I threw this into my code, and went along my merry way:
`entries = Hash.new(Hash.new([]))`. A few days later, I came back
to this code to finish it and realised that something was awry with the
values I was getting out of the `Hash`. It took me a while to figure it
out because the values were all integers being summed together and I was
never looking or setting the values directly.

However, eventually I saw a pattern — many of the values were the same.
For instance, given the following assignments, I should expect
`[ruby]p entries[153][:monday] #=> [12.hours]`:

``` ruby
entries[153][:monday]  << 12.hours
entries[153][:tuesday] << 3.hours
entries[87][:monday]   << 7.hours
entries[87][:tuesday]  << 2.5.hours
```

However, what I found was the following:

``` ruby
p entries[153][:monday]  #=> [12.hours, 3.hours, 7.hours, 2.5.hours]
p entries[153][:tuesday] #=> [12.hours, 3.hours, 7.hours, 2.5.hours]
p entries[87][:monday]   #=> [12.hours, 3.hours, 7.hours, 2.5.hours]
p entries[87][:tuesday]  #=> [12.hours, 3.hours, 7.hours, 2.5.hours]

# And in fact:
p entries[:any][:thing]  #=> [12.hours, 3.hours, 7.hours, 2.5.hours]
```

Let that soak in for a second. `Hash.new({})` uses the **same
instance** of the internal `Array` as the default value for each
of the inner-most keys. It also doesn’t set the key you want so
`p entries` still printed `{}`. In retrospect, it is
blatantly obvious that it does this, but the ramifications are still
huge.

The way to get the intended effect is of course to use the block syntax
of `[ruby]Hash.new` which, while much uglier, definitely works as
expected:

``` ruby
entries = Hash.new {|hash, key| hash[key] = Hash.new {|h, k| h[k] = [] }}
entries[153][:monday]  << 12.hours
entries[153][:tuesday] << 3.hours
entries[87][:monday]   << 7.hours
entries[87][:tuesday]  << 2.5.hours

p entries #=> {87=>{:monday=>[7.hours], :tuesday=>[2.5.hours]}, 153=>{:monday=>[12.hours], :tuesday=>[3.hours]}}
```

Wow. Obvious, but good to remember. I am sure most of you have had this
issue before, but I think it is still worthy of mention…