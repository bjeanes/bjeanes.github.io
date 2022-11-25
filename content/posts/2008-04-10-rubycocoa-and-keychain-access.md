+++
aliases = ["2008/rubycocoa-and-keychain-access", "2008/04/rubycocoa-and-keychain-access"]
date = 2008-04-10T12:00:00Z
slug = "rubycocoa-and-keychain-access"
title = "RubyCocoa and Keychain access"
updated = 2011-08-28T21:21:57Z
+++

There have been a few posts asking how to access and manipulate the
Keychain from within RubyCocoa but no answers supplied. As someone who
is just getting into RubyCocoa (and Cocoa all together for that matter),
I thought I’d document once and for all the process I took to get it
working.

I am running on Leopard and while 10.5 is supposed to ship with
BridgeSupport for most frameworks, the Security framework must have been
left out. After playing around with using other frameworks (such as the
AddressBook framework) and seeing how they were used, I did a bit of
digging in the Apple Developer Documentation (incredible resource for
anyone starting out) and stumbled across this gem: [Generate Framework
Metadata](http://developer.apple.com/documentation/Cocoa/Conceptual/RubyPythonCocoa/Articles/GenerateFrameworkMetadata.html#//apple_ref/doc/uid/TP40005426)

After reading this, I was accessing the keychain within 5 minutes.
Here’s how:

1.  Run the following commands (first one will take some time):

    ``` console
    gen_bridge_metadata -f Security -o Security.bridgesupport
    mkdir -p /Library/BridgeSupport
    mv Security.bridgesupport /Library/BridgeSupport
    ```
2. Open an IRB session to test it:

``` irb
>> require 'osx/cocoa'
=> true
>> OSX.require_framework 'Security'
=> true
>> defined? OSX::SecKeychainAddGenericPassword()
=> "method"
```
Hoorah! Two steps! Well, one because the second one was just testing…
Now go make password-able RubyCocoa application! I am not sure how to
pull this off with deployable apps but I suggest your apps could ship
with the file or run those commands on first-run or if the
Security.bridgesupport file doesn’t exist. Note: I have not tested
either of those scenarios, but I would guess the latter would be more
reliable for cross-version development (i.e. Tiger + Leopard)

### How to use these methods

``` ruby
# Require needed libraries

require 'osx/cocoa'
include OSX
require_framework 'Security'

# Set up some relevant variables

service = "Test Service"
account = "MyUsername"
password = "MyPassword"

# Add password to default keychain (first param)

SecKeychainAddGenericPassword(nil, service.length, service, account.length, account, password.length, password, nil)

# Finding a password in default keychain (first param)

status, *password = SecKeychainFindGenericPassword(nil, service.length, service, account.length, account)

# Password-related data
password_length = password.shift # => 10
password_data = password.shift # OSX::ObjcPtr object
password = password_data.bytestr(password_length)

# The last item is another OSX::ObjcPtr. I haven't figured 
# out how to cast or use this yet but will post when I've
# figured it out
keychain_item = password.shift

# Yay it works! You can also check the password exists
# in the Keychain Access utility. I've confirmed that
# this works
puts password # => "MyPassword"
```

### [UPDATE 4/8/08]: Below is a fix for the REXML bug some people have
been getting (myself included)

There is good news for anyone that has been getting the following error:

``` text
undefined local variable or method `trans’ for <UNDEFINED> … </>:REXML::Document
Usage: gen_bridge_metadata [options] <headers…>
Use the `-h’ flag or consult gen_bridge_metadata(1) for help.
```

There is a very simple fix. This is caused by a typo in the source of
REXML. It’s such a massive bug I can’t believe it made it into the
release of Ruby but i did some poking around and it looks like they
renamed a method and didn’t change all references to it.

There are two ways to fix it—patching REXML; and patching `gen_bridge_metadata`.

Personally I prefer patching the generator for peace of mind and have submitted the patch at
[http://bridgesupport.macosforge.org/trac/ticket/1](http://bridgesupport.macosforge.org/trac/ticket/1),
but I’ll show both fixes.

#### Patching gen*bridge*metadata

1.  Open `/usr/bin/gen_bridge_metadata and find the method definition for
    `generate_xml`
2.  Replace `0` in the `xml_document.write()` with `-1` (both occurrences)
3.  Save!

#### Patching REXML

1.  Open
    `/System/Library/Frameworks/Ruby.framework/Versions/1.8/usr/lib/ruby/1.8/rexml/document.rb`
2.  Find the line that has `if trans` and change it to `if transitive`
3.  Save!

