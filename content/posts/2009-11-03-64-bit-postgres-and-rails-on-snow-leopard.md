+++
aliases = ["2009/64-bit-postgres-and-rails-on-snow-leopard", "2009/11/64-bit-postgres-and-rails-on-snow-leopard"]
date = 2009-11-03T05:12:00Z
slug = "64-bit-postgres-and-rails-on-snow-leopard"
title = "64-bit Postgres and Rails on Snow Leopard "
updated = 2011-08-28T20:59:41Z
+++

### Dependencies

Install the GEOS and PROJ frameworks from
[here](http://www.kyngchaos.com/software:frameworks)

### Postgres

#### Download and Compile

``` bash
http://ftp2.au.postgresql.org/pub/postgresql/source/v8.4.1/postgresql-8.4.1.tar.bz2 | tar xjf -
cd postgresql-8.4.1
./configure
make && sudo make install
```

The files are now all installed in the right places, onwards to making
them usable!

#### Create a `postgres` User and Group

This is adjusted from the instructions in the comments
[here](http://www.postgresql.org/docs/8.3/interactive/postgres-user.html)).

``` bash
# Find unused Group ID and User ID:
export GROUPID=`sudo dscl . -list /Groups PrimaryGroupID | ruby -e 'puts STDIN.read.scan(/\d+/m).map{|i|i.to_i}.uniq.sort.last.succ'`
export USERID=`sudo dscl . -list /Users UniqueID | ruby -e 'puts STDIN.read.scan(/\d+/m).map{|i|i.to_i}.uniq.sort.last.succ'`

# Create group:
sudo dscl . -create /Groups/_postgres
sudo dscl . -create /Groups/_postgres PrimaryGroupID $GROUPID
sudo dscl . -append /Groups/_postgres RecordName postgres

# Create user:
sudo dscl . -create /Users/_postgres
sudo dscl . -create /Users/_postgres UniqueID $USERID
sudo dscl . -create /Users/_postgres PrimaryGroupID $GROUPID
sudo dscl . -create /Users/_postgres UserShell /bin/bash
sudo dscl . -create /Users/_postgres RealName "PostgreSQL Server"
sudo dscl . -create /Users/_postgres NFSHomeDirectory /usr/local/pgsql
sudo dscl . -append /Users/_postgres RecordName postgres
```

That’s the clean way of making a user (so it doesn’t show up in your
Accounts preference pane, etc.

#### Clean up

``` bash
sudo touch /var/log/psql.log
sudo mkdir /usr/local/pgsql/data
sudo chown -R postgres:postgres /usr/local/pgsql/data /var/log/psql.log

# Put Postgres in your path
export $PATH="/usr/local/pgsql/bin:$PATH"
sudo sh -c 'echo /usr/local/pgsql/bin > /etc/paths.d/pgsql'
```

Thanks to Apple we have a nice way of setting the PATH across the entire
system so every app should now know how to find the Postgres binaries
and the data directory is now set up to store the database files — now,
we just need to create them.

#### Initialise the Database

``` bash
# Become the postgres user
sudo su - postgres

# If this throws next commant throws an error about shared memory, 
# try putting "these lines":http://gist.github.com/224815 into 
# /etc/sysctl.conf, rebooting, and trying again:
initdb -D /usr/local/pgsql/data -E UTF8

# Start the database
postgres -D /usr/local/pgsql/data >/var/log/psql.log 2>&1 &

# Create a test database and check you can connect to it
createdb test
psql test

# Just create a superuser for general dev tasks
createuser postgres

# we don't want to be the postgres user anymore
exit
```

Your done with the Postgres setup now!

### PostGIS

``` bash 
curl http://postgis.refractions.net/download/postgis-1.3.7SVN.tar.gz | tar xzf -
cd postgis-1.3.7SVN
./configure --with-geosconfig=/Library/Frameworks/GEOS.framework/unix/bin/geos-config --with-projdir=/Library/Frameworks/PROJ.framework/unix
make && sudo make install
```

Phew! That one was easy.

### Postgres Ruby Gem

This is the part that everyone cares about the most. If this works your
Rails apps should start working like a charm.

``` bash
sudo env ARCHFLAGS='-arch x86_64' gem install pg
```