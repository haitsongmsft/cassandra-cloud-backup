Cassandra Backup and Restore with Azure Storage's file share
====================
Shell script for creating and managing Cassandra Backups using Azure storage's file share
## Features
- Take snapshot backups
- Copy Incremental backup files
- Compress with gzip or bzip2 to save space
- Prune old incremental and snapshot files
- Execute Dry Run mode to identify target files

## Requirements
Azure cli installed (if not, follow instruction like this in linux):
    sudo apt-get install apt-transport-https lsb-release software-properties-common -y
    AZ_REPO=$(lsb_release -cs)
    echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" | \
        sudo tee /etc/apt/sources.list.d/azure-cli.list

    sudo apt-key --keyring /etc/apt/trusted.gpg.d/Microsoft.gpg adv \
         --keyserver packages.microsoft.com \
         --recv-keys BC528686B50D79E339D3721CEB3E94ADBE1229CF

    sudo apt-get update
    sudo apt-get install azure-cli

Linux system with BASH shell. 
Cassandra 2+


## Usage
./cassandra-cloud-backup.sh [ options ] < command> 

### Examples
  Take a full snapshot, gzip compress it with nice=15,
  upload into the azure file storage, and clear old incremental and snapshot files
  ./cassandra-azure-backup.sh -A myazstorageacct -K mystoragekey -zCc -N 15 backup

  Do a dry run of a full snapshot with verbose output and
  create list of files that would have been copied
  ./cassandra-azure-backup.sh -A myazstorageacct -K mystoragekey -vn backup

  Backup and bzip2 compress copies of the most recent incremental
  backup files since the last incremental backup
  ./cassandra-azure-backup.sh -A myazstorageacct -K mystoragekey -ji backup

  Restore a backup without prompting from specified bucket path and keep the old files locally
  ./cassandra-azure-backup.sh -A myazstorageacct -K mystoragekey -b host01/snpsht/2016-01-20_18-57/ -fk restore

  Restore a specific backup to a custom CASSANDRA_HOME directory with secure credentials in
  password.txt file with Cassandra running as a Linux service name cass
  ./cassandra-azure-backup.sh -A myazstorageacct -K mystoragekey -b host01/snpsht/2016-01-20_18-57/ \
   -y /opt/cass/conf/cassandra.yaml -H /opt/cass -U password.txt -S cass restore

  List inventory of available backups stored in Azure Cloud Store
  ./cassandra-azure-backup.sh -A myazstorageacct -K mystoragekey inventory
  ./cassandra-azure-backup.sh -U mysetting.txt inventory

### Commands:

- backup        ---        Backup the Cassandra Node based on passed in options
- restore        ---      Restore the Cassandra Node from a specific snapshot backup  
- inventory      ---       List available backups
- commands      ---        List available commands
- options       ---        list available options

### Options:
Flags:
  -a, --alt-hostname
    Specify an alternate server name to be used in the bucket path construction. Used
    to create or retrieve backups from different servers

  -A  --az-storage
    Azure storage account name, it can be put in a file to be used in the shell via -U, or environment variable AZURE_STORAGE_ACCOUNT

  -B, backup
    Default action is to take a backup

  -b --backup-path
    The file base path of the azure file share within the azure storage account. When doing backups, a fileshare
    with name of "cassandradump" is created. The base path is host name, and snapshots storaged in the folder 
    of "hostname/snpsht/date_snapshotid". In the resore case, it is expected to pass "hostname/date_snapshotid" for This
    parameter

  -c, --clear-old-ss
    Clear any old SnapShots taken prior to this backup run to save space
    additionally will clear any old incremental backup files taken immediately
    following a successful snapshot. this option does nothing with the -i flag

  -C, --clear-old-inc
    Clear any old incremental backups taken prior to the the current snapshot

  -d, --backupdir
    The directory in which to store the backup files, be sure that this directory
    has enough space and the appropriate permissions

  -D, --download-only
    During a restore this will only download the target files from azure file share

  -e, --endpoint-cqlsh
    Endpoint IP of cqlsh, defaults to the first up node reported by nodetool. This is used when we 
    need to obtain a copy of schema via running cqlsh.

  -f, --force
    Used to force the restore without confirmation prompt

  -h, --help
    Print this help message.

  -H, --home-dir
    This is the $CASSANDRA_HOME directory and is only used if the data_directories,
    commitlog_directory, or the saved_caches_directory values cannot be parsed out of the
    yaml file.

  -i, --incremental
    Copy the incremental backup files and do not take a snapshot. Can only
    be run when compression is enabled with -z or -j

  -j, --bzip
    Compresses the backup files with bzip2 prior to pushing to Azure Cloud Storage
    This option will use additional local disk space set the --target-gz-dir
    to use an alternate disk location if free space is an issue

  -k, --keep-old
    Set this flag on restore to keep a local copy of the old data files
    Set this flag on backup to keep a local copy of the compressed backup, schema dump,
    and token ring

  -K, --storage-key 
    Either the sas token, or the account key for uploading to your storage accounts, it can be in a file with -U, or 
    put in an environment Variable of AZURE_STORAGE_KEY

  -l, --log-dir
    Activate logging to file 'CassandraBackup${DATE}.log' from stdout
    Include an optional directory path to write the file
    Default path is /var/log/cassandra

  -L, --inc-commit-logs
    Add commit logs to the backup archive. WARNING: This option can cause the script to
    fail an active server as the files roll over

  -n, --noop
    Will attempt a dry run and verify all the settings are correct

  -N, --nice
    Set the process priority, default 10

  -p
    The Cassandra User Password if required for security

  -r,  restore
    Restore a backup, requires a --backup-path and optional --backupdir

  -s, --split-size
    Split the resulting tar archive into the configured size in Megabytes, default 100M

  -S, --service-name
    Specify the service name for cassandra, default is cassandra use to stop and start service

  -T, --target-gz-dir
    Override the directory to save compressed files in case compression is used
    default is --backupdir/compressed, also used to decompress for restore

  -u
    The Cassandra User account if required for security

  -U, --auth-file
    A file that contains authentication credentials for cqlsh, nodetool, and azure connection string consisting of
    these lines:
      CASSANDRA_USER=username
      CASSANDRA_PASS=password
      AZURE_STORAGE_ACCOUNT=AzureStorageAccountName
      AZURE_STORAGE_KEY=accountKeyIfUsed
      AZURE_STORAGE_SAS_TOKEN=sasConnectionStringIfUsed

  -v, --verbose
    When provided will print additional information to log file

  -w, --with-caches
    For posterity's sake, to save the read caches in a backup use this flag, although it
    likely represents a waste of space

  -y, --yaml
    Path to the Cassandra yaml configuration file
    default: /etc/cassandra/cassandra.yaml

  -z, --zip
    Compresses the backup files with gzip prior to pushing to Azure Cloud Storage
    This option will use additional local disk space set the --target-gz-dir
    to use an alternate disk location if free space is an issue

 
###Cron Examples
- Full gzip compressed snapshot every day at 1:30 am with nice level 10

`30 1 * * * /path_to_scripts/cassandra-cloud-backup.sh -z -N10 -d /var/lib/cassandra/backups > /var/log/cassandra/$(date +\%Y\%m\%d\%H\%M\%S)-fbackup.log 2>&1`

- Incremental gzip compressed backups copied every hour nice level 10

`0 * * * * /path_to_scripts/cassandra-cloud-backup.sh -N10 -vjiz -d /var/lib/cassandra/backups > /var/log/cassandra/$(date +\%Y\%m\%d\%H\%M\%S)-ibackup.log 2>&1`

### Notes

The script must be run with sufficient privileges to be able to stop/start processes and create/delete directories and files within the working directories.

The restore command is designed to perform a simple restore of a full snapshot. In the event that you want to restore incremental backups you should start by restoring the last full snapshot prior to your target incremental backup file and manually move the files from each incremental backup in chronological order leading up to the target incremental backup file.  The schema dump is included in the snapshot backups, but if necessary it must also be restored manually.

Snapshots are taken at the system level, the script currently does not support backup or restore of an individual keyspace or columnfamily. 

In order to enable incremental backups, the `incremental_backups` option has to be set to true in the cassandra.yaml file.

###License
 Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at
      http://www.apache.org/licenses/LICENSE-2.0
Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS-IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.  See the License for the specific language governing permissions and limitations under the License.
