---
author: "Karthik M"
title: "MySQL DB backup using Bash script"
date: "2018-01-05"
tags:
- linux
- bash
- database
- mysql
- mariadb
---

Backups are important for any data. Taking DB backups for MySQL/MariaDB is a cinch with the following script.

## Bash script
Create a file called backup_script.sh. Replace the credentials and path to suit your system.

```
#!/bin/bash

USER='username'
PASSWORD='password'
DATABASE='ALL_DB'
# Directory should always be mentioned with a trailing slash
DIRECTORY='/path/for/db_backup/'
DOW=$(date +"%A")

# Delete backups which are 7 days old
rm -f $DIRECTORY$DATABASE-$DOW.sql.gz > /dev/null 2>&1

#Backup the DB and Compress it
/usr/bin/mysqldump --user=$USER --password=$PASSWORD --host=$HOST --all-databases --skip-lock-tables > $DIRECTORY$DATABASE-$DOW.sql
/bin/gzip $DIRECTORY$DATABASE-$DOW.sql
```
The script will create/keep in total 7 backups, with the day of the week as its suffix. Escape `$` sign if its used in the
`PASSWORD` variable.

Provide execute permission to the script
```
$ chmod 755 backup_script.sh
```

Make sure to verify the backup-file is generated by executing the script:
```
$ ./backup_script.sh
```
## CRON Job
Add the following line to your crontab. This will automatically execute a backup daily at 0400 hours.
```
0 4 * * * /bin/sh /path/to/backup_script.sh >> /dev/null 2>&1
```
