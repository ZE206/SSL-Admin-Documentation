## Task 4

```bash
#creating users
sudo useradd -m -s /bin/bash exam_1
sudo useradd -m -s /bin/bash exam_2
sudo useradd -m -s /bin/bash exam_3
sudo useradd -m -s /bin/bash examaudit
sudo useradd -m -s /bin/bash -G sudo examadmin

#setting up passwrod
sudo passwd exam_1
sudo passwd exam_2
sudo passwd exam_3
sudo passwd examaudit
sudo passwd examadmin

#verify users created
cat /etc/passwd | grep exam
```

### Home directory security
```bash
sudo chmod 700 /home/exam1 /home/exam2 /home/exam3
#this puts a rule that owner of that directory can only

#which is what chmod 700 is about 
```

### Disk Quota
To prevent any single user from exhausting the system's storage (a common issue in 1GB RAM/small disk VMs), I implemented hard and soft disk quotas.
```bash
# i added usrquota in /etc/fstab

sudo quotacheck -cum -f
sudo quotaon 
sudo setquota -u exam_1 1024000 1228800 0 0 /
sudo setquota -u exam_2 1024000 1228800 0 0 /
sudo setquota -u exam_3 1024000 1228800 0 0 /
```
Soft limit: 1GB (warning), Hard limit: 1.2GB (absolute max, writes fail beyond this). examadmin and examaudit have no quota limits.

### Backup Script
I developed a Bash script to archive examinee data. The script is designed for security, ensuring only the examadmin can access the archives.
```bash
#creating backup script
nano automate.sh

#!/bin/bash
BACKUP_DIR="/var/backups/exam_users"
DATE=$(date+%Y%m%d_%H%M%S)
mkdir -p $BACKUP_DIR
tar -czf $BACKUP_DIR/exam_backup_$DATE.tar.gz /home/exam_1 /home/exam_2 /home/exam_3
chmod 600 $BACKUP_DIR/exam_backup_$DATE.tar.gz
chown examadmin:examadmin $BACKUP_DIR/exam_backup_$DATE.tar.gz
echo "Backup completed: $BACKUP_DIR/exam_backup_$DATE.tar.gz"
```

### cron job
To ensure data persistence without manual intervention, I scheduled the backup to run daily during off-peak hours (2:00 AM).
```bash
0 2 * * * /usr/local/bin/backup_exam_users.sh
```
### verification

User Check: ```bash grep "exam" /etc/passwd```

Quota Check: ```bash repquota -au```

Backup Check: ```bash ls -lh /var/backups/exam_users/ ```