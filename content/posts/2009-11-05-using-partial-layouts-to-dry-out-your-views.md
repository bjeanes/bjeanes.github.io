+++
aliases = ["2009/using-partial-layouts-to-dry-out-your-views", "2009/11/using-partial-layouts-to-dry-out-your-views"]
date = 2009-11-05T00:59:00Z
slug = "using-partial-layouts-to-dry-out-your-views"
title = "Using partial layouts to DRY out your views"
updated = 2011-08-28T21:28:55Z

[taxonomies]
tags = ["rails"]
+++

The other day we got some HTML from the designer of one of our clients to implement as the new UI for an app we are
building at work. Overall the HTML was good but implementing it started to feel like I was repeating myself.

For example, around all the apps form the client’s design had a rounded white box, the HTML of which went something like
this:

``` html
<div class="content-wht">
    <div class="content-wht-inside">
        <fieldset>
            <div class="frow">
                <label>Currency:</label>
                <select class="fs"><option>EUR</option></select>
            </div>

            <div class="frow frlast">
                <label>File:</label>
                <input type="file" name="" />
            </div>
        </fieldset>
    </div>
</div>

<div class="cround" align="center">
    <img src="/images/round-corners-bottom.gif" alt="" />
</div>
```

It’s a lot of `div` tags to have to repeat around every form that has this style. Luckily a nice feature of Rails
partials can help out.  Create a shared partial file that will act as a layout like this:

``` erb
<!-- app/views/shared/_fieldset.html.erb -->
<div class="content-wht">
    <div class="content-wht-inside">
        <fieldset>
            <%= yield %>
        </fieldset>
    </div>
</div>

<div class="cround" align="center">
    <img src="/images/round-corners-bottom.gif" alt="" />
</div>
```

Then in your views that need to have this white box, simply wrap your content in that layout like so:

``` erb
<% render :layout => 'shared/fieldset' do %>
  <div class="frow">
      <label>Currency:</label>
      <select class="fs"><option>EUR</option></select>
  </div>

  <div class="frow frlast">
      <label>File:</label>
      <input type="file" name="" />
  </div>
<% end %>

<!-- Or, if the inner content is already in a partial -->

<%= render, :partial => 'form', :layout => 'shared/fieldset' %>
```
