---
title: "Automatically open the last page for failed scenarios"
layout: 'post'
author: bo
---

When writing new Cucumber scenarios, developers will usually start with
a failing scenario, incrementally making it pass, step by step. If this
is a common workflow for you, you’ll probably have noticed a trend of
having to re-run it each time a new step is failing, adding something
like `Then show me the page` after the last
passing step to see why the story is currently failing.

[Chendo](http://chendo.net) and I came up with a nifty little solution
in just a few lines that really helps this process of seeing what webrat
sees when a scenario fails. Throw this snippet of code in a file in your
`features/support/` directory:

``` ruby
After do |scenario|
  if scenario.respond_to? && scenario.status == :failed
    save_and_open_page
  end
end
```
