---
{"publish":true,"title":"MySQL Replication","created":"2025-08-29","modified":"2025-08-30T00:20:41.313-04:00","published":"2025-08-29","tags":["databases","mysql","replication"],"cssclasses":""}
---

MySQL Supports built-in replication in 3 major methods.

# Master-Master Replication

Install mysql on master 1 and slave 1 and configure the network services on both systems similar to:

## Step 1

~~~~~
Master 1/Slave 2 ip: 192.168.16.4
Master 2/Slave 1 ip: 192.168.16.5
~~~~~

## Step 2

On Master 1, make changes in my.cnf

~~~~~
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
old_passwords=1
log-bin
binlog-do-db=<database name>  # input the database which should be replicated
binlog-ignore-db=mysql        # input the database that should be ignored for replication
binlog-ignore-db=test

server-id=1

[mysql.server]
user=mysql
basedir=/var/lib

[mysqld_safe]
err-log=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
~~~~~

## Step 3

On master 1, create a replication slave account in mysql.

~~~~~
mysql> grant replication slave on *.* to 'replication'@192.168.16.5 identified by 'slave';
~~~~~

and restart the mysql master1.

## Step 4

Now edit my.cnf on Slave1/Master2:

~~~~~
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
old_passwords=1

server-id=2

master-host = 192.168.16.4
master-user = replication
master-password = slave
master-port = 3306

[mysql.server]
user=mysql
basedir=/var/lib

[mysqld_safe]
err-log=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
~~~~~

## Step 5

Restart mysql slave1/master2:

~~~~~
mysql> start slave;
mysql> show slave status\G;
*************************** 1. row ***************************

             Slave_IO_State: Waiting for master to send event
                Master_Host: 192.168.16.4
                Master_User: replica
                Master_Port: 3306
              Connect_Retry: 60
            Master_Log_File: MASTERMYSQL01-bin.000009
        Read_Master_Log_Pos: 4
             Relay_Log_File: MASTERMYSQL02-relay-bin.000015              Relay_Log_Pos: 3630
      Relay_Master_Log_File: MASTERMYSQL01-bin.000009
           Slave_IO_Running: Yes
          Slave_SQL_Running: Yes
            Replicate_Do_DB:
        Replicate_Ignore_DB:
         Replicate_Do_Table:
     Replicate_Ignore_Table:
    Replicate_Wild_Do_Table:
Replicate_Wild_Ignore_Table:
                 Last_Errno: 0
                 Last_Error:
               Skip_Counter: 0
        Exec_Master_Log_Pos: 4
            Relay_Log_Space: 3630
            Until_Condition: None
             Until_Log_File:
              Until_Log_Pos: 0
         Master_SSL_Allowed: No
         Master_SSL_CA_File:
         Master_SSL_CA_Path:
            Master_SSL_Cert:
          Master_SSL_Cipher:
             Master_SSL_Key:
      Seconds_Behind_Master: 1519187

1 row in set (0.00 sec)
~~~~~

Above highlighted rows must indicate related log files and Slave_IO_Running and Slave_SQL_Running to be YES.

## Step 6

On master1/slave2:

~~~~~
mysql> show master status;
+------------------------+----------+--------------+------------------+
| File                   | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------------+----------+--------------+------------------+
|MysqlMYSQL01-bin.000008 |      410 | adam         |                  |
+------------------------+----------+--------------+------------------+
1 row in set (0.00 sec)
~~~~~

The above scenario is for master-slave, now we will create a slave master scenario for the same systems and it will work as master-master.

## Step 7

On Master2/Slave1, edit my.cnf adding the master entries into it:

~~~~~
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
# Default to using old password format for compatibility with mysql 3.x
# clients (those using the mysqlclient10 compatibility package).
old_passwords=1
server-id=2

master-host = 192.168.16.4
master-user = replication
master-password = slave
master-port = 3306

log-bin                     #information for becoming master added
binlog-do-db=adam

[mysql.server]
user=mysql
basedir=/var/lib

[mysqld_safe]
err-log=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
~~~~~

## Step 8

Edit my.cnf on Master1/Slave2 for information of it's joining master:

~~~~~
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

# Default to using old password format for compatibility with mysql 3.x
# clients (those using the mysqlclient10 compatibility package).
old_passwords=1

log-bin
binlog-do-db=adam
binlog-ignore-db=mysql
binlog-ignore-db=test

server-id=1
#information for becoming slave.
master-host = 192.168.16.5
master-user = replication
master-password = slave2
master-port = 3306

[mysql.server]
user=mysqlbasedir=/var/lib
~~~~~
