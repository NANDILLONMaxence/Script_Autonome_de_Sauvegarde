### Presentation and Explanation of Autonomous Backup Script

This script aims to facilitate the automatic backup of a PostgreSQL database (in this example, the Zabbix database) to a remote TrueNAS server. It follows the following steps:

### Source Server (Debian)

#### Prerequisites
- **Installation of Utilities**:
  - `postgresql-client`: Command-line tool to interact with PostgreSQL `You can do this with any type of DB!`.
  - `rsync`: File synchronization utility.
  - `sshpass`: Allows providing a password when SSHing.
  - `cron`: Task scheduling system.

#### Step 1: Installation of mysqldump Utility
```bash
sudo apt update
sudo apt install postgresql-client rsync sshpass cron -y
```

#### Step 2: Creation of a Backup Script
Create a script (`backup.sh`) to automate the backup process:
```bash
#!/bin/bash

# Setting Parameters
DB_USER="zabbix"
DB_PASS="BabouchedeMichel"
DB_NAME="zabbix"
BACKUP_DIR="/home/max/Install_auto_Zabbix/"
DATE=$(date +"%Y-%m-%d_%H-%M-%S")
BACKUP_FILE="$BACKUP_DIR/$DB_NAME-$DATE.sql"

# Creating Backup
PGPASSWORD=$DB_PASS pg_dump -U $DB_USER -d $DB_NAME -h localhost -w > $BACKUP_FILE

# Transfer to TrueNAS using rsync with the sauv account
rsync -avz --progress -e "sshpass -p 'root' ssh -o StrictHostKeyChecking=no" $BACKUP_FILE sauv@192.168.1.128:/mnt/test/SAUV-01
```

### Explanation of Script Parts:

- **DB_USER**: Database user.
- **DB_PASS**: Database user's password.
- **DB_NAME**: The database name.
- **BACKUP_DIR**: Directory where the backup will temporarily be stored on your Debian server.
- **PGPASSWORD=$DB_PASS pg_dump -U $DB_USER -d $DB_NAME > $BACKUP_FILE**: Exports the PostgreSQL database to a SQL file.
- **rsync**:
  - `-avz --progress -e "sshpass -p 'backup-server-user-password' ssh -o StrictHostKeyChecking=no" $BACKUP_FILE backup-server-user@backup-server-ip:/path/to/backup/folder`: Uses rsync to transfer the backup file to your backup server.

#### Step 3: Making the Script Executable
```bash
chmod +x backup.sh
```

### Backup Server (TrueNAS)

#### Step 1: Share Configuration
Ensure you have a share configured on your TrueNAS, accessible from your Debian server. Note the share path (e.g., `/mnt/TrueNASBackup`).

#### Step 2: Mounting the Share on Debian Server
Mount the TrueNAS share on your Debian server so the script can copy the backups there:
```bash
sudo mkdir /mnt/truenas_backup
sudo mount -t cifs //trueNAS_ip_address/share_name /mnt/truenas_backup -o username=your_username,password=your_password,uid=$(id -u),gid=$(id -g),vers=3.0
```

### Explanation of Mount Options:
- **-t cifs**: Specifies the filesystem type as CIFS.
- **//192.168.1.128/sauv-01**: The IP address of your TrueNAS and the name of the share you want to mount.
- **/mnt/truenas_backup**: The mount directory on your Debian system.
- **-o**: Specifies mount options:
- **username=sauv**: The username to connect to the CIFS share.
- **password=root**: The password to connect to the CIFS share (in this example, it's "root", make sure to replace with the correct password).
- **uid=$(id -u)**: Ensures the logged-in user is the owner of the mount point (optional, can be replaced with the desired user's UID).
- **gid=$(id -g)**: Ensures the logged-in group is the owner of the mount point (optional, can be replaced with the desired group's GID).
- **vers=3.0**: Specifies the version of the CIFS protocol to use. This may be necessary depending on the version of TrueNAS you're using.

#### Step 3: Granting Permissions on Backup Folder
```bash
sudo chmod -R 770 /path/to/SAUV-01
```

### Testing the Script
Run the script to check its functionality:
```bash
./backup.sh
```
Then, verify that the operation completed successfully.

### Step 4: Automating Backups with Cron
Add a cron task to execute the backup script at regular intervals:
```bash
crontab -e
```
Add the following line for a daily backup at 2 AM, for example:
```
0 2 * * * /path/to/your/script/backup.sh
```
or

A weekly backup at 2 AM every Sunday:
```
0 2 * * 0 /path/to/your/script/backup.sh
```

Also, verify that the operation completed successfully after automation.

**Checking the Configuration**:
- To check if the Cron task was configured correctly, you can display your crontab using the following command:
  ```bash
  crontab -l
  ```

- You should see the line you added, confirming that the backup task is scheduled to run automatically every 7 days.

#### These steps should enable you to set up an automatic backup system for your PostgreSQL database to your TrueNAS server, ensuring the security and availability of your data.
----
