---
title: "Iterative HTML5 - Turning a comma-separated list into a semantic list"
layout: 'post'
author: bo
---

In the process of going through the redesign of [http://mocra.com](http://mocra.com), I keep finding better ways to make
the HTML more semantic.

[Jack Chen](http://chendo.net) and I have been turning the design into 
HTML over the last few days and we’ve been striving to be as semantic as
possible by taking advantage of new HTML5 elements and by separating the
content further from the design code with all the new CSS3 selectors.

Here’s an example of something we just did to make the HTML cleaner and
more meaningful, whilst still maintaining the look and style.

Here’s what we are styling:

![image](http://img.skitch.com/20091112-qujhy29f4e5mkpradfd3hb9hnj.jpg)

Usually your HTML (or HAML in this case) would look something like this:

``` haml
%footer
  %aside
    = link_to 'Dr Nic Williams', 'http://drnicwilliams.com/'
    ,
    = link_to 'Bodaniel Jeanes', 'http://bjeanes.com/'
    ,
    = link_to 'Jack Chen', 'http://chendo.net/'
    ,                      
    = link_to 'Ryan Bigg', 'http://frozenplague.net/'
```

However, we can make this **list** of people more semantic by using an
`<ul>` element and some CSS magic.

Here’s what the new HTML (HAML here) would look like:

``` haml
%footer
  %aside
    %ul
      %li= link_to 'Dr Nic Williams', 'http://drnicwilliams.com/'
      %li= link_to 'Bodaniel Jeanes', 'http://bjeanes.com/'
      %li= link_to 'Jack Chen',       'http://chendo.net/'
      %li= link_to 'Ryan Bigg',       'http://frozenplague.net/'
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

