+++
aliases = ["2009/how-to-prepare-your-mac-for-resale", "2009/07/how-to-prepare-your-mac-for-resale"]
date = 2009-07-01T03:30:00Z
slug = "how-to-prepare-your-mac-for-resale"
title = "How to Prepare Your Mac for Resale"
updated = 2011-08-28T21:15:51Z

[taxonomies]
tags = ["mac"]
+++

First thing is first: you want to re-install Mac OS X (this guide requires Leopard). Insert your OS X disk and boot off
it. Make sure you do a "Clean Install" so as not to leave any of your files on the system.

When the install is finished, your computer will restart and you’ll see the traditional OS X welcome screen and the
setup assistant. Go through all the settings, choosing anything just to get the system booted up.  However, make sure
you take note of the user account login you use. For this guide, I will assume login of `temp`, and if you don’t have a
good reason not to, you should do the same.

At this point you want to download any Apple updates and install them, as well as install any bundled software you plan
to advertise the machine coming with. When all your installs are finished and the machine is as you want it for the
customer/recipient, shut down the machine.

Next, turn the machine back on but as soon as you hear the Mac chime noise, press and hold ⌘S until you are taken to a
console (white text on black screen — looks real nerdy and sci-fi).

The next step is to run all the commands in the code block below.

_NOTE: If you didn’t use `temp` as your username you created in the setup assistant, make sure you replace **all**
instances of `temp` with your actual username._

``` bash
# Mount the file system so we can modify it
/sbin/fsck -fy
/sbin/mount -uw /

# Activate directory services
/bin/launchctl load /System/Library/LaunchDaemons/com.apple.DirectoryServices.plist &

# Delete all traces of the temporary account
/usr/bin/dscl . -delete /Users/temp
/usr/bin/dscl . -delete /Groups/admin GroupMembership temp
/bin/rm -rf /Users/temp

# Get rid of a few other files
/bin/rm -R /Library/Preferences
/bin/rm -f /root/.bash_history

# Now we make it appear as though the initial setup assistant
# still needs to run
/bin/rm /var/db/.AppleSetupDone

# Reboot and check that it seems like
# it is a new mac, then turn off your computer
# and sell it!
/sbin/reboot
```
