---
title: "Using fields_for with Nested Attributes, Calling it Multiple Times"
layout: 'post'
author: bo
---

There has been quite an annoying problem that has been bugging
[@chendo](http://twitter.com/chendo) and I today.

In short, [`fields_for`](http://api.rubyonrails.org/classes/ActionView/Helpers/FormHelper.html#M001895) when used with `accepts_nested_attributes_for`does not reset it’s index when it’s called multiple times for the same attribute.

Our scenario was the following:

We had a 3 levels of hierarchical data that we had to display in a
`<table>`. Because of the nature of the data we were
displaying one level as columns and the other level as rows.
Unfortunately, HTML mandates that we group everything by rows — each
column is really just a single cell. This means that we have to iterate
over the “column” values not just once, but once for every row.

Now, `fields_for` takes care of naming your
`<input>` tags magically with the correct indexes so that
your models can accept the params directly and magic happens. However,
evidently calling `fields_for` multiple times was not
part of the original intention, and it seems resetting the index after
each call was neglected.

`chendo couldn't see a cleaner way of doing this so we added our own option to reset the index. We didn't just override the behaviour in case there is a good reason to leave it, but here is our code:

``` ruby
class ActionView::Helpers::FormBuilder
  def fields_for_with_nested_attributes_with_index_reset(association_name, args, block)
    if args.last.is_a?(Hash) && args.last[:reset_index]
      @nested_child_index = nil
    end

    fields_for_without_nested_attributes_without_index_reset(association_name, args, block)
  end

  alias_method_change :fields_for_with_nested_attributes, :index_reset
end
```

