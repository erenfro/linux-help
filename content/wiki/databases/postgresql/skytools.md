---
{"publish":true,"title":"PostgreSQL and Skytools","created":"2025-08-29","modified":"2025-08-30T00:22:54.407-04:00","published":"2025-08-29","tags":["databases","postgresql","skytools"],"cssclasses":""}
---

Skype made a series of tools that are extremely useful for PostgreSQL. One of them I use is walmgr.py. It allows you to not just make a complete backup of the raw database for fast recovery, but also allows you to setup a fully warm standby instance of PostgreSQL, collecting WALs (Write Ahead Logs), and allows, through some customized approaches, PITR (Point In Time Recovery). I will cover all these in this document.

# Setting up Skytools walmgr

First thing to do is setup walmgr itself, which is has a few steps involved. Setting up the ini files for both master and slave, running the walmgr setup and then making the first backup. Here are example master.ini and slave.ini files:

**master.ini:**

~~~~~
[wal-master]
job_name        = live_walgmr_master
logfile         = /var/lib/postgresql/log/wal-master.log
use_skylog      = 0

master_db       = dbname=template1
master_data     = /var/lib/postgresql/8.3/data
master_config       = /var/lib/postgresql/8.3/data/postgresql.conf
# set this only if you can afford database restarts during setup and stop.
#master_restart_cmd = /etc/init.d/postgresql-8.3 restart

slave           = __hostname__:/var/lib/postgresql/walshipping
slave_config        = /var/lib/postgresql/conf/slave.ini

completed_wals      = %(slave)s/logs.complete
partial_wals        = %(slave)s/logs.partial
full_backup     = %(slave)s/data.master
config_backup       = %(slave)s/config.backup

# syncdaemon update frequency
loop_delay      = 10.0
# use record based shipping available since 8.2
use_xlog_functions  = 0
# pass -z flag to rsync
compression     = 0

# periodic sync
#command_interval   = 600
#periodic_command   = /var/lib/postgresql/walshipping/periodic.sh
~~~~~

**slave.ini:**

~~~~~
[wal-slave]
job_name        = slave1_walmgr_slave
logfile         = /var/lib/postgresql/log/wal-slave.log
use_skylog      = 0

slave_data      = /var/lib/postgresql/8.3/data
slave_bin       = /usr/bin
slave_stop_cmd      = sudo /etc/init.d/postgresql-8.3 stop
slave_start_cmd     = sudo /etc/init.d/postgresql-8.3 start
slave_config_dir    = /var/lib/postgresql/8.3/data

slave           = /var/lib/postgresql/walshipping
completed_wals      = %(slave)s/logs.complete
partial_wals        = %(slave)s/logs.partial
full_backup     = %(slave)s/data.master
config_backup       = %(slave)s/config.backup

keep_backups        = 4
archive_command     =
~~~~~

Once these are setup, you will notice the directory structures are pretty standard. The next step of configuration is to, on the live database, create the initial basic structure it would use:

~~~~~
mkdir -p /var/lib/postgresql/conf
chown postgres:postgres /var/lib/postgresql/conf
~~~~~

And on the slave server(s):

~~~~~
mkdir -p /var/lib/postgresql/conf
mkdir -p /var/lib/postgresql/logs.complete
mkdir -p /var/lib/postgresql/logs.partial
mkdir -p /var/lib/postgresql/data.master
mkdir -p /var/lib/postgresql/config.backup
chown postgres:postgres /var/lib/postgresql/conf
chown postgres:postgres /var/lib/postgresql/logs.complete
chown postgres:postgres /var/lib/postgresql/logs.partial
chown postgres:postgres /var/lib/postgresql/data.master
chown postgres:postgres /var/lib/postgresql/config.backup
~~~~~

# Making a new WALs Backup

From the Live database server and as the user postgres as well, execute:

~~~~~
walmgr.py /var/lib/postgresql/conf/master.ini backup
~~~~~

This will rotate, on the slave server, the walshipping/data.master to data.master.0 automatically and populate it with a brand new base backup. This will not effect usability of the live server except the fact it will slightly increase load on the disks of the live database. It is fairly safe to run even in busy hours.

# Restoring from a WALs Backup

To use walmgr to restore the database, as the postgres user, NOT ROOT, use:

~~~~~
walmgr.py /var/lib/postgresql/conf/slave.ini restore data.master
~~~~~

It is important to use data.master at the end because if you do not, it will MOVE the current base image into /var/lib/postgresql/8.3/data instead of copy the current snapshot.

# PITR, WALs, and Skytools

## Listing WALs Backups

~~~~~
walmgr.py /var/lib/postgresql/conf/slave.ini listbackups
~~~~~

This will show which available backups there are to restore from. At this time, only 2 weeks worth are being stored. Keep in mind that prior backups than the current data.master will likely be archived and compressed to save space as much as possible, and walmgr is not currently equiped to handle these at the moment.
