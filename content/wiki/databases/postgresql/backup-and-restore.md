---
{"publish":true,"title":"PostgreSQL Backup and Restore","created":"2025-08-29","modified":"2025-08-30T00:22:19.289-04:00","published":"2025-08-29","tags":["databases","postgresql","backup","restore"],"cssclasses":""}
---

The most basic way to backup and restore PostgreSQL databases is with the pg_dump and pg_restore tools. This process is extremely taxing to the database server in both backing up and restoring, but it is a complete backup of the schema, data, and everything.

PostgreSQL 8.4 and later has actually made it so pg_dump and pg_restore can do multiple databases at the same time instead of just one at a time to improve performance greatly.

First thing is to have a ~/.pgpass with the details needed to authenticate to the database. This file is in the format of:

~~~~~
hostname:port:database:username:password
~~~~~

The first 3 fields can be wildcarded with *. The file itself must be named .pgpass in your home directory, and chmod 0600 or it will refuse it.

To make a backup of a particular database, you can use pg_dump like so:

~~~~~
pg_dump --format c --file filename.pgdump dbname
~~~~~

This dumps just the one database into a gzip-compressed file. That is what the --format does, custom, which essentially means gzip.

Otherwise, if you want to do a full backup of every database you can use pg_dumpall:

~~~~~
pg_dumpall --file filename.sql dbname
~~~~~

This, unlike the pg_dump with --format c, will export straight SQL directly to the file which is why I set it to use filename.sql instead of pgdump, to classify it as such.

There are other options, but in generality, this is all you need in basics. For help on them in further detail, it's as simple as using either of the following, or both, of these:

~~~~~
pg_dump --help
pg_dumpall --help
~~~~~
