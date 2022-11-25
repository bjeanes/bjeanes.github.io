+++
aliases = ["2009/using-alterego-with-activerecord", "2009/07/using-alterego-with-activerecord"]
date = 2009-07-03T06:32:00Z
slug = "using-alterego-with-activerecord"
title = "Using AlterEgo with ActiveRecord"
updated = 2011-08-28T21:15:08Z
+++

Today after finding the gem AlterEgo and falling in love with the API,
[@chendo](http://twitter.com/chendo) and I came to a sad realisation that its beauty was skin-deep. It seems it is not ActiveRecord-aware and overrides `#state` and `#state=` on your models. This means that your `state` column goes
untouched and your models are always in the default state.

To get around this, @chendo and I hacked together this little monkey patch which does two things:

1. It adds question methods for each of your defined states. So if you have states: `active`, `inactive`, and `banned`
on your `User` model, your instances will have `#active?`, `#inactive?`, and `#banned?` 
2.  It properly sets and gets the state of the row and loads it into AlterEgo.

Here is the code:

``` ruby
module AlterEgo
  def self.included(base)
    base.instance_eval do
      
      # This will allow us to define a question method for each state in a class
      # For instance if user has states :active and :inactive, methods active? and inactive?
      # will be defined when first called
      def method_missing_with_states(method, *args, &block)
        begin
          method_missing_without_states(method, *args, &block)
        rescue NoMethodError => e
          state_found = false
          result = nil
          
          base.states.keys.each do |state|
            if method.to_s == "#{state}?" && !state_found
              self.class.send :define_method, "#{state}?" do
                self.state == state
              end
              state_found = true
              result = self.__send__(method)
            end
          end
          raise(e) unless state_found
          result
        end
      end
      alias_method_chain :method_missing, :states
      
      # state setter to write to database
      define_method :state= do |value|
        write_attribute(:state, value.to_s)
      end

      # state getter to read from database
      define_method :state_with_active_record do
        (read_attribute(:state) || (self.state = state_without_active_record)).to_sym
      end
      alias_method_chain :state, :active_record
    end
  end
end
```
