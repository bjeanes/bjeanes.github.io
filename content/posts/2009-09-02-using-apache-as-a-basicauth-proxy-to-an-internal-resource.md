+++
aliases = ["2009/using-apache-as-a-basicauth-proxy-to-an-internal-resource", "2009/09/using-apache-as-a-basicauth-proxy-to-an-internal-resource"]
date = 2009-09-02T12:53:00Z
slug = "using-apache-as-a-basicauth-proxy-to-an-internal-resource"
title = "Using Apache as a BasicAuth proxy to an internal resource"
updated = 2011-08-28T21:05:42Z
+++

This article isn’t named ideally so if you have a better title, tell me.

This is the scenario: we have a simple HTTP resource on the internal network at [Mocra](http://mocra.com). This resource
is a closed source tool which is great except that it has one drawback — there are only two security modes: basic
authentication on, or basic authentication off.  There is only one username/password and you can’t control when or by
whom it is required.

Now, my preferred behaviour here is to be able to give out multiple login/passwords and also to only require
authentication **outside** of our local network. I set out to get this working.

I devised this solution a few months ago (which is why it’s using Apache at all) but I thought it might be of some use
to someone. I created an Apache VirtualHost to act as a proxy between this resource and its clients. It looks like this:

```
<VirtualHost *:80>
  # This virtual host proxies requests for the internal resource
  ServerName external.resource.com

  # Set auth header to base64 encoded 'username:password'
  # where username and password are the credentials for internal resource
  RequestHeader set Authorization "Basic dXNlcm5hbWU6cGFzc3dvcmQ="

  ProxyPass /resource_path/ http://192.168.1.123:1337/resource_path/
  ProxyPassReverse /resource_path/ http://192.168.1.123:1337/resource_path/

  <Location />
    Order allow,deny
    Allow from 192.168.1.0/24 # our local subnet
    Allow from 127.0.0.1 # duh
    Allow from 1.3.3.7 # our external router IP

    AuthType basic
    AuthName "Our Internal Resource"
    # where all the external login/passwords are
    AuthUserFile /var/www/.htpasswd
    Require valid-user

    # this is the secret sauce that allows EITHER
    # local access or Basic Auth access
    Satisfy Any
  </Location>

  RewriteEngine on
  RewriteRule ^/?$ http://external.resource.com:80/resource_path/ [R]
</VirtualHost>
```
