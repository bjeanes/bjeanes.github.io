---
title: "Using accepts_nested_attributes_for When the Child Object validates_presence_of Parent"
layout: 'post'
author: bo
---

I came across an interesting dilemma today in Rails 2.3 whilst trying to
use `accepts_nested_attributes_for`.

The models I have were:

``` ruby
class Organization < ActiveRecord::Base
  has_many :users
  accepts_nested_attributes_for :users
end

class User < ActiveRecord::Base
  belongs_to :organization
  validates_presence_of :organization
end
```

I had an “Organization” creation form which allowed an administrator to
create an Organization and an initial user for it. However, even though
I had filled in all the required fields to pass the validations I was
constantly getting the following error: `Organization can not be blank`.

Digging deeper I found that when an object is added to an association
via the `association.build` method, the parent object
isn’t actually set:

``` irb
>> o = Organization.new(:name => "Test")
=> #<Organization id: nil, name: "Test">
>> u = o.users.build(:name => "Test User")  
=> #<User id: nil, organization_id: nil, name: "Test User">
>> u.organization
=> nil
```

Apparently this flaw also applies to
`accepts_nested_attributes_for`. When I send through the
User `params` through from my form normally with the
`Organization` params, it creates the
`User` object in the `users` association
as you’d expect, but raises a validation error because the
`User` instance doesn’t recognise that it is has a parent
`Organization`

My quick 5 minute fix was to modify the `Organization`
model thusly:

``` ruby
class Organization < ActiveRecord::Base
  has_many :users
  accepts_nested_attributes_for :users

  def users_attributes_with_self_assignment=(attributes)
    self.users_attributes_without_self_assignment = attributes
    users.each { |u| u.organization = self }
  end
  alias_method_chain :users_attributes=, :self_assignment
end
```

Related Rails bugs seem to be
[here](https://rails.lighthouseapp.com/projects/8994-ruby-on-rails/tickets/1943)
and
[here](https://rails.lighthouseapp.com/projects/8994-ruby-on-rails/tickets/2815).
[This
patch](https://rails.lighthouseapp.com/projects/8994/tickets/1619-patch-support-for-inverse-option-in-associations)
will also solve the problem and is currently [in
edge](http://github.com/rails/rails/commit/ccea98389abbf150b886c9f964b1def47f00f237),
but didn’t make it in time for Rails 2.3.

