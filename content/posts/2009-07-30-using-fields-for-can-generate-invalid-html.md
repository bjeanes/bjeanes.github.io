+++
aliases = ["2009/using-fields-for-can-generate-invalid-html", "2009/07/using-fields-for-can-generate-invalid-html"]
date = 2009-07-30T03:47:00Z
slug = "using-fields-for-can-generate-invalid-html"
title = "Using fields_for can generate invalid HTML"
updated = 2011-08-28T20:39:47Z

[taxonomies]
tags = ["rails"]
+++

Just a quick post so show you how `fields_for` form helper can cause invalid HTML when used in combination with
`accepts_nested_attributes_for` in your views.

If you are for any reason editing/creating your child objects in a tabular form, your view might look something like
this:

``` erb
<% form_for(@todo_list) do |f| %>
  <%= f.label      :name %>
  <%= f.text_field :name %>

  <h3>To Do Items:</h3>
  <table>
    <thead>
      <tr>
        <th>To Do Item</th>
        <th>Due Date</th>
        <th>Done?</th>
      </tr>
    </thead>
    <tbody>
      <% f.fields_for :items do |item| %>
        <tr>
          <td><%= item.text_field :content %></td>
          <td><%= item.text_field :due_date %></td>
          <td><%= item.check_box  :done %></td>
        </tr>
      <% end %>
    </tbody>
  </table>

  <%= f.submit 'Save' %>
<% end %>
```

At first glance this looks fine. However, when the `ToDoList` model has `accepts_nested_attributes_for :items`, the
`fields_for` helper also outputs a hidden field for each existing `Item` instance with it’s ID.

This means that we get the following HTML:

``` html
<form action="/to_do_lists/1" class="edit_to_do_list" id="edit_to_do_list_1" method="post">
  <div style="margin:0;padding:0;display:inline">
    <input name="_method" type="hidden" value="put" />
    <input name="authenticity_token" type="hidden" value="CE2jyRoewbwsqu4eX8AFWFqsgrMfCN35Jzy6b43MhsA=" />
  </div>
  <label for="to_do_list_name">Name</label>
  <input id="to_do_list_name" name="to_do_list[name]" size="30" type="text" value="Sup" />

  <h3>To Do Items:</h3>
  <table>
    <thead>
      <tr>
        <th>To Do Item</th>
        <th>Due Date</th>
        <th>Done?</th>
      </tr>
    </thead>
    <tbody>
      <input id="to_do_list_items_attributes_0_id" name="to_do_list[items_attributes][0][id]" type="hidden" value="1" />
        <tr>
          <td><input id="to_do_list_items_attributes_0_content" name="to_do_list[items_attributes][0][content]" size="30" type="text" value="arst" /></td>
          <td><input id="to_do_list_items_attributes_0_due_date" name="to_do_list[items_attributes][0][due_date]" size="30" type="text" value="2009-07-30" /></td>
          <td><input name="to_do_list[items_attributes][0][done]" type="hidden" value="0" /><input id="to_do_list_items_attributes_0_done" name="to_do_list[items_attributes][0][done]" type="checkbox" value="1" /></td>
        </tr>
      <input id="to_do_list_items_attributes_1_id" name="to_do_list[items_attributes][1][id]" type="hidden" value="2" />
        <tr>
          <td><input id="to_do_list_items_attributes_1_content" name="to_do_list[items_attributes][1][content]" size="30" type="text" value="arst" /></td>
          <td><input id="to_do_list_items_attributes_1_due_date" name="to_do_list[items_attributes][1][due_date]" size="30" type="text" value="2009-07-30" /></td>
          <td><input name="to_do_list[items_attributes][1][done]" type="hidden" value="0" /><input id="to_do_list_items_attributes_1_done" name="to_do_list[items_attributes][1][done]" type="checkbox" value="1" /></td>
        </tr>
    </tbody>
  </table>

  <input id="to_do_list_submit" name="commit" type="submit" value="Save" />
</form>
```

The keen eye will notice the hidden `<input>` tags that are direct children of the `tbody`:

``` html
<tbody>
  <input id="to_do_list_items_attributes_0_id" name="to_do_list[items_attributes][0][id]" type="hidden" value="1" />
    <tr><!-- clip! --></tr>
  <input id="to_do_list_items_attributes_1_id" name="to_do_list[items_attributes][1][id]" type="hidden" value="2" />
    <tr><!-- clip! --></tr>
</tbody>
```

This is not a valid place for an `<input>` tag.

I’ve put an example project demonstrating this on [my GitHub](http://github.com/bjeanes/fields_for_invalid_html)
