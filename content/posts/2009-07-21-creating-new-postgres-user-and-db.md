+++
aliases = ["2009/creating-new-postgres-user-and-db", "2009/07/creating-new-postgres-user-and-db"]
date = 2009-07-21T03:25:00Z
slug = "creating-new-postgres-user-and-db"
title = "Creating new Postgres User and DB"
updated = 2011-08-28T21:06:33Z
+++

This post is mostly a reminder for me in the future. However, others may
find it useful also.

I love Postgres but more often than not I get frustrated and confused
trying to get authentication working with a new database. By looking
through the Deprec source code, I pulled out these two lines of code and
so far they haven’t failed me:

``` bash
su - postgres -c "createuser -P -D -A -E $DBUSER"
su - postgres -c "createdb -O $DBUSER $DBNAME"
```

Of course, replace `$DBUSER` and
`$DBNAME` with your desired username and database
name, or define them as shell variables. The
`$DBUSER` creates that user as the owner of the
database (so it has permission to read/write etc).