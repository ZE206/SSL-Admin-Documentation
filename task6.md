## Task 6

## installing mariadb
```bash
sudo apt install mariadb-server -y
sudo systemctl enable mariadb
sudo systemctl start mariadb
```

## secure installation
mysql_secure_installation is a shell script that address the  common security gaps  of mysql or mariadb
```bash
sudo mysql_secure_installation
```
questions:
    root password set
    anonymous users removed ->they r risk
    remote root login disabled->root should never be accessed over network
    test db removed ->
    privileges table removed

## database and user setup

```bash
CREATE DATABASE secure_onboarding;
CREATE USER 'dbuser'@'localhost' IDENTIFIED BY 'DbUser@1234';
GRANT SELECT, INSERT, UPDATE, DELETE ON secure_onboarding.* TO 'dbuser'@'localhost';
FLUSH PRIVILEGES;
```

To bind to localhost only, we need to set bind-address=127.0.0.1 in # /etc/mysql/mariadb.conf.d/50-server.cnf. This address is of 'lo' interface which is a loopback interface.

ss -tlnp | grep 3306


## setting backup script

```bash
#!/bin/bash
BACKUP_DIR="/var/backups/mysql"
DATE=$(date +%Y%m%d_%H%M%S)
mkdir -p $BACKUP_DIR
mysqldump -u root secure_onboarding | gzip > $BACKUP_DIR/db_backup_$DATE.sql.gz
chmod 600 $BACKUP_DIR/db_backup_$DATE.sql.gz
echo "DB Backup completed: $BACKUP_DIR/db_backup_$DATE.sql.gz"
```

make backup script executable
```bash
sudo chmod 700 /usr/local/bin/backup_db.sh
sudo chown root:root /usr/local/bin/backup_db.sh
```


## setting up cron tab

```bash
sudo crontab -e
```

add this line to bottom 
```bash
0 3 * * * /usr/local/bin/backup_db.sh
```

