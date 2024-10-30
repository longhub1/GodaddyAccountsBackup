# GodaddyAccountsBackup

To automate weekly backups of your GoDaddy environment (like the `public_html` directory) to your local environment, you can use a script to download and save these backups. Here’s a step-by-step guide:

### Steps to Set Up Automated Weekly Backups

1. **Set Up SSH Access to Your GoDaddy Hosting**  
   - First, ensure that SSH access is enabled on your GoDaddy hosting account.
   - If you’re using cPanel, enable SSH access from the cPanel interface under "SSH Access."

2. **Create a Backup Script on Your Local Environment**  
   This script will:
   - Connect to your GoDaddy hosting via SSH.
   - Compress the `public_html` directory into a `.zip` or `.tar.gz` file.
   - Transfer the compressed file to your local system.

   Here's an example of a Bash script that you can schedule to run weekly:

   ```bash
   #!/bin/bash

   # Configurations
   REMOTE_USER="your_godaddy_username"
   REMOTE_HOST="your_godaddy_host_ip"
   REMOTE_DIR="public_html"
   LOCAL_BACKUP_DIR="/path/to/local/backup"
   BACKUP_NAME="backup_$(date +%Y%m%d).tar.gz"

   # Compress the remote directory
   ssh $REMOTE_USER@$REMOTE_HOST "tar -czf /home/$REMOTE_USER/$BACKUP_NAME -C /home/$REMOTE_USER $REMOTE_DIR"

   # Download the backup to the local directory
   scp $REMOTE_USER@$REMOTE_HOST:/home/$REMOTE_USER/$BACKUP_NAME $LOCAL_BACKUP_DIR/

   # Clean up remote backup file
   ssh $REMOTE_USER@$REMOTE_HOST "rm /home/$REMOTE_USER/$BACKUP_NAME"

   # Keep only one week's worth of backups (7 days)
   find $LOCAL_BACKUP_DIR -type f -name "backup_*.tar.gz" -mtime +7 -exec rm {} \;

   echo "Backup completed and old backups cleaned."
   ```

   - Replace `your_godaddy_username`, `your_godaddy_host_ip`, and `/path/to/local/backup` with your specific details.
   - This script will:
     - Compress the `public_html` directory remotely.
     - Download the compressed backup file to a specified local directory.
     - Remove backup files older than 7 days from the local environment.

3. **Set Up a Weekly Cron Job on Your Local Machine**  
   To automate this process weekly, set up a cron job on your local system (if you're on a Unix-based OS like Linux or macOS):
   ```bash
   crontab -e
   ```
   Add the following line to schedule the script to run every Sunday at midnight:
   ```bash
   0 0 * * 0 /path/to/your_backup_script.sh
   ```
   Make sure to replace `/path/to/your_backup_script.sh` with the actual path to your script file.

4. **Testing and Permissions**  
   - Make sure your script file is executable:
     ```bash
     chmod +x /path/to/your_backup_script.sh
     ```
   - Test the script manually first by running it to ensure it’s working as expected.

This setup will ensure you have an automated weekly backup stored locally, retaining only the last 7 days’ worth of backups.
