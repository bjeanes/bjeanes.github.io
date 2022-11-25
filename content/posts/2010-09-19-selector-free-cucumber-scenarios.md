+++
aliases = ["2010/selector-free-cucumber-scenarios", "2010/09/selector-free-cucumber-scenarios"]
date = 2010-09-19T13:53:00Z
slug = "selector-free-cucumber-scenarios"
title = "Selector-Free Cucumber Scenarios"
updated = 2013-03-14T15:37:12Z
+++

<small>(a Serbo-Croation translation by Anja Skrbaa is available [here](http://science.webhostinggeeks.com/selektor-free))</small>

I’ve been using Cucumber since pretty much the first day I heard about
it. I’ve worked on a lot of projects that have relied on it’s presence
for reliable development. Therefore, I’ve put a lot of effort into
perfecting my Cucumber infrastructure to make this fantastic tool even
better. I’m going to share one such morsel of code that makes developing
with Cucumber even greater.

## The Problem

I’ve worked on a lot of web applications and, as I’m sure many of you
know, quite often the development of a web application is focussed on
the functionality foremost, and the interface and style is incorporated
later. It may be that the client doesn’t yet know the feel they want for
their project or that they want to focus their budget towards
prototyping the application first.

This is fine, except for the fact that changing the HTML and CSS of a
web application after a lot of functionality has been developed is a
fantastic way to break all your integration tests.

This is particularly true if you have scenarios like the following
contrived one:

``` cucumber
When I fill in "Username" with "bjeanes" within ".main-panel form#signup-form"
And I press "Sign up!" within ".main-panel form#signup-form"
Then I should see "You have successfully signed up" within ".flash.notice"
```

The problem is, of course, that designers might change the HTML that
used to be `.main-panel form#signup-form` into something sexier and more
semantic.

## The Solution

This problem is not unlike an already-solved one; we’ve all moved away
from hardcoding URLs like "/users/new" into our views and Cucumber
scenarios and replacing them with `new_user_path` and `the signup page`,
respectively.

So why not apply the same formula that `paths.rb` uses for removing URLs
from scenarios to our situation with selectors?

Here’s what I add to all new projects:

``` ruby
# I'm in features/step_definitions/web_ext_steps.rb

When /^(.*) within ([^:"]+)$/ do |step, scope|
  with_scope(selector_for(scope)) do
    When step
  end
end

# Multi-line version of above
When /^(.*) within ([^:"]+):$/ do |step, scope, table_or_string|
  with_scope(selector_for(scope)) do
    When "#{step}:", table_or_string
  end
end  
```

And:

``` ruby
# I'm in features/support/selectors.rb

module HtmlSelectorsHelper
  def selector_for(scope)
    case scope
    when /the body/
      "html > body"
    else
      raise "Can't find mapping from \"#{scope}\" to a selector.\n" +
        "Now, go and add a mapping in #{__FILE__}"
    end
  end
end

World(HtmlSelectorsHelper)
```

## Applying the Solution

My previous example of the flawed Cucumber scenario now becomes:

``` cucumber
When I fill in "Username" with "bjeanes" within the sign up form
And I press "Sign up!" within the sign up form
Then I should see "You have successfully signed up" within the notice flash
```

And the `selectors.rb` case statement gets the following additions:

``` ruby
case
  # ...
  
  when /the sign up form/
    ".main-panel form#signup-form"
  when /the (notice|error|info) flash/
    ".flash.#{$1}"
  
  # ...
end
```

Notice how the scenario now identifies things on our page by their
semantic identifiers, not by brittle CSS or XPath locations which are
prone to change. As a bonus, now <del>if</del> when they do change, the paths only need to be updated in a single
location in our Cucumber test suite!

## Patching Cucumber

I feel pretty strongly that CSS and XPath don’t belong in our feature
files because not only does it encourage brittle tests (as shown above),
but also because those selectors are entirely irrelevant to end users,
and that’s kind of the main point of using a natural language DSL to
describe our integration tests, i.e. putting on the user shoes.

<del>
I’d really like to patch this back into Cucumber, and I entirely plan to
do so, providing I get the time.

</del>
I got the time, and here is my [pull
request](http://github.com/aslakhellesoy/cucumber-rails/pull/63) to have
it merged.

## Expanding the Solution

You’ll note my `HtmlSelectorsHelper` module only accommodates CSS
selectors. That’s only because I have never needed XPath in this
context. It’d be very simple to modify my examples to do so, though,
with a combination of multiple return values and a splat. That’s an
exercise for the reader (or me if I end up patching Cucumber).

## Final Word

I apologise for the length of this article, but I congratulate you for
making it all the way through it!

I now have so many blog post ideas lined up that I’ve had to create a
new category in Things.app just to hold them all. This means that I’ll
be striving to get a few more posts done and out the door in the next
few weeks, including a performance comparison of different data
encapsulations for web application APIs on the iPhone (i.e. is it better
to use Plists, JSON, or XML?) and a post on why I think there should be
8 RESTful actions, not the 7 that Rails prescribes by default.