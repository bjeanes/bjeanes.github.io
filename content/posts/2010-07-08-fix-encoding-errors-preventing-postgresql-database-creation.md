+++
aliases = ["2010/fix-encoding-errors-preventing-postgresql-database-creation", "2010/07/fix-encoding-errors-preventing-postgresql-database-creation"]
date = 2010-07-08T09:20:00Z
slug = "fix-encoding-errors-preventing-postgresql-database-creation"
title = "Fix encoding errors preventing PostgreSQL database creation"
updated = 2011-08-28T20:49:28Z
+++

I recently was setting up a new VPS on Linode and I got the following
error when trying to create a new database:

``` text
ERROR: new encoding is incompatible with the encoding of the template database
HINT: Use the same encoding as in the template database, or use template0 as template.
```

This error is related to the locale of the system when Postgres was
installed. If you run the commands in my [previous
post](/2010/07/08/fix-locale-errors-on-linux) before installing
Postgres, you should avoid these errors altogether.

If Postgres has already been installed, however, you can run the
following commands (assuming Postgres version 8.4) to reinitialise the
database with the correct encoding:

``` console
pg_dropcluster --stop 8.4 main
pg_createcluster --start -e UTF-8 8.4 main
```

Note: this will erase all databases and reset your postgres
configuration so it is only really useful when you are first setting up
your database, or have taken appropriate measures to be able to restore
your data.
