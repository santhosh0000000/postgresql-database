 #!/bin/bash

# List all of the Psql databases that you want to backup in here,
# Backup path
backup_path="/tmp"

# Full PostgreSQL dump (All Databases)
postgres_backup="yes"
postgres_user="ssk"
postgres_pass="sand"
postgres_database="ssk"
postgres_host="localhost"
postgres_port="5432"

# Send an email the result of the backup process
# You should have sendmail or postfix installed
# Send Backup?  Would you like the backup emailed to you?
# Set to "y" if you do
send_email="no"
subject="Backup is done"
email_to="sahul.s@open-v.net"

# Unix Commands
 
mail=/bin/mail

# Main variables
color='\033[0;36m'
color_fail='\033[0;31m'
nc='\033[0m'
hostname=$(hostname -s)
date_now=$(date +"%Y-%m-%d %H:%M:%S")

path_date=$(hostname -s)_$(date +"%Y-%m-%d-%H-%M-%S")
mkdir -p $backup_path/Backup/$path_date 2>> $log_file
echo -e "\n ${color}--- $date_now Backup started. \n${nc}"
echo "$date_now Backup started." >> $log_file



# PostgreSQL backup
if [ $postgres_backup = "yes" ]
then
    # Creating ~/.pgpass for PostgreSQL password
    # PostgreSQL does not support inline password
    # Know better solution? Let me know.
    USERNAME=`whoami`
    CUR_DATE=$(date +"%Y-%m-%d-%H-%M-%S")
    if [ $USERNAME = "root" ]
    then
        echo "$postgres_host:$postgres_port:$postgres_database:$postgres_user:$postgres_pass" > /root/.pgpass
        chmod 600 /root/.pgpass
    else
        echo "$postgres_host:$postgres_port:$postgres_database:$postgres_user:$postgres_pass" > /home/$USERNAME/.pgpass
        chmod 600 /home/$USERNAME/.pgpass
    fi

    echo -e "\n ${color}--- $date_now PostgreSQL backup enabled, backing up: \n${nc}"
    echo "$date_now PostgreSQL backup enabled, backing up" >> $log_file
    # Using ionice for PostgreSQL dump
    ionice -c 3 pg_dump -p $postgres_port -h $postgres_host -Fc -U $postgres_user $postgres_database > ${backup_path}/Backup/${path_date}/Postgres_Full_Dump_${path_date}.dump | tee -a $log_file
    if [ $? -eq 0 ]
    then
        echo -e "\n ${color}--- $date_now PostgreSQL backup completed. \n${nc}"
        echo "$date_now PostgreSQL backup completed" >> $log_file
    fi
fi

sleep 1

# Create TAR file
echo -e "\n ${color}--- $date_now Creating TAR file located in $backup_path/Full_Backup_$path_date.tar.bz2 \n${nc}"
echo "$date_now Creating TAR file located in $backup_path/Full_Backup_$path_date.tar.bz2" >> $log_file
tar -cjf $backup_path/Full_Backup_${path_date}.tar.bz2 $backup_path/Backup/$path_date 2> /dev/null
rm -rf $backup_path/Backup/
final_archive="Full_Backup_${path_date}.tar.bz2"

sleep 1

# Send a simple email notification
if [ $send_email = "yes" ]
then
   
     $mail -s "$subject : $postgres_user" $email_to <Full_Backup_${path_date}.tar.bz2
fi

# print end status message
echo "."
echo "."
echo "."
echo "... Backup SUCCESS!"
date
echo ""
echo "Delete old files !"
find Full_Backup_${path_date} -mtime +30 -type f -delete
echo ""

# And we're done
ls -l ${backup_path}
echo "Dump Complete!"
exit 
