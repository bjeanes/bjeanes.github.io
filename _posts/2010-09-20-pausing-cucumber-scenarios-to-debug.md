---
title: "Pausing Cucumber Scenarios to Debug"
layout: 'post'
author: bo
---

Continuing what I started with my [post last
light](http://bjeanes.com/2010/09/19/selector-free-cucumber-scenarios),
I’ve got another tip for developing productively when using Cucumber.

I’m going to show you a simple way to pause cucumber stories in order to
investigate the current state of your application at that point.

``` ruby
AfterStep('@pause') do
  print "Press Return to continue"
  STDIN.getc
end
```

Then just tag any Cucumber feature or scenario with `@pause`. After the
first step is run, the scenario will stop running and you can do
whatever you like before proceeding.

My favourite tactic is to combine the `@pause` tag with `@selenium` tag
to have WebDriver stop driving FireFox for a moment and let me click
around in the application or inspect the HTML/CSS at that point.

My next post will be about enabling FireBug in the "WebDriver" profile
to help inspecting the web page when you’ve paused a scenario.

