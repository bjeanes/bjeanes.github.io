+++
aliases = ["2009/iterative-html5-turning-a-comma-separated-list-into-a-semantic-list", "2009/11/iterative-html5-turning-a-comma-separated-list-into-a-semantic-list"]
date = 2009-11-12T06:42:00Z
slug = "iterative-html5-turning-a-comma-separated-list-into-a-semantic-list"
title = "Iterative HTML5 - Turning a comma-separated list into a semantic list"
updated = 2011-08-28T21:04:52Z

[taxonomies]
tags = ["css"]
+++

In the process of going through the redesign of our company website, I keep finding better ways to make the HTML more
semantic.

[Jack Chen](https://chen.do) and I have been turning the design into HTML over the last few days and we’ve been
striving to be as semantic as possible by taking advantage of new HTML5 elements and by separating the content further
from the design code with all the new CSS3 selectors.

Here’s an example of something we just did to make the HTML cleaner and more meaningful, whilst still maintaining the
look and style.

Usually your HTML (or HAML in this case) would look something like this:

``` haml
%footer
  %aside
    = link_to 'Dr Nic Williams', 'https://github.com/drnic'
    ,
    = link_to 'Bo Jeanes', 'http://bjeanes.com/'
    ,
    = link_to 'Jack Chen', 'https://chen.do/'
    ,
    = link_to 'Ryan Bigg', 'https://ryanbigg.com/'
```

However, we can make this **list** of people more semantic by using an `<ul>` element and some CSS magic.

Here’s what the new HTML (HAML here) would look like:

``` haml
%footer
  %aside
    %ul
      %li= link_to 'Dr Nic Williams', 'https://github.com/drnic'
      %li= link_to 'Bo Jeanes', 'https://bjeanes.com/'
      %li= link_to 'Jack Chen', 'https://chen.do/'
      %li= link_to 'Ryan Bigg', 'https://ryanbigg.com/'
```

And the CSS (Sass here):

``` sass
body
  footer
    aside
      padding-top: 6px
      float: left

      ul
        list-style: none
        margin: 0
        padding: 0

        li
          display: inline

          &:after
            content: ","

          &:last-child:after
            content: ""
```
