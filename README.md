# Using Duplicacy to backup securely to free Storj cloud storage

## Introduction

[Duplicacy](https://duplicacy.com) is state-of-the-art backup tool that has extensive cloud support. It also supports local disks and remote sftp servers.     

Duplicacy is available as a web-based GUI or as a command line tool. It's extremely quick and uses [Lock Free Deduplication](https://github.com/gilbertchen/duplicacy/wiki/Lock-Free-Deduplication).

The software does require a license but the **command-line interface** (CLI) version is free for personal use.  
  
We’ll be using the CLI version for this tutorial and backing up to [Storj](https://www.storj.io/) cloud storage. Duplicacy can backup to any S3 compatible cloud storage. 

Here is a list of supported [backends](https://forum.duplicacy.com/t/supported-storage-backends/1107) including sftp.


## Installation    

### Packages

Pre-compiled binaries are available for Linux, macOS, and Windows directly from the Duplicacy [GitHub](https://github.com/gilbertchen/duplicacy/releases) repository.

Download the latest version for your system. Currently the latest version is `version 2.7.2`.
~~~
$ wget https://github.com/gilbertchen/duplicacy/releases/download/v2.7.2/duplicacy_linux_x64_2.7.2
~~~

Make the file executable.
~~~
$ chmod +x duplicacy_linux_x64_2.7.2
~~~

Rename the file to **duplicacy**.
~~~
$ mv duplicacy_linux_x64_2.7.2 duplicacy
~~~

Change the file permissions.
~~~
$ chmod 755 duplicacy
~~~

Move the file to the **/usr/bin** directory.
~~~
$ sudo mv duplicacy /usr/bin/
~~~

## Preparing a new repository

The directory that you want to backup is called a `repository`. The location where your backup will be stored is called the `storage url`.     

The **duplicacy init** command is used to initialize the storage location and the backup directory.
~~~
duplicacy init [command options] <snapshot id> <storage url>
~~~

The `<snapshot id>` refers to the name you want to call your backup job.    
The `<storage url>` refers to the storage location for your backup job.

The **duplicacy init** command has multiple options, but we'll only be using the following:

~~~
-encrypt, -e          encrypt the storage with a password
-storage-name <name>  assign a name to the storage
-repository <path>    initialize a new repository at the specified path
~~~

We will be initializing the repository using the **duplicacy init** command and these options in the next section.

## StorJ cloud storage


[Storj](https://www.storj.io/) is an S3 compatible cloud storage provider that offers 150 GB of free storage. **No credit card is required**.

In order to backup data via Storj, you must first create a bucket and S3 credentials from your account page.

1. Sign up for a free account to get started.
2. Create a new storage bucket.
3. Create your S3 credentials.

Using the **Access key ID** and **Secret access key** from your S3 credentials you will initialize your repository and bucket.

StorJ uses a [Gateway endpoint](https://docs.aws.amazon.com/vpc/latest/privatelink/gateway-endpoints.html) for establishing connections:
~~~
gateway.storjshare.io 
~~~
You need to also specify the regional location for your account. For me it would be the following:
~~~
us-east-1
~~~

Specify the **regional location**, **gateway endpoint**, and **bucket name** as part of the `<storage url>`.

An example should make this easier to understand.

* I created a storage bucket on StorJ named `photosbackup`.
* I am backing up a directory in my home directory named `Photos`. 
* I am using `storjphotos` as the storage name.
* I am also using `photosbackup` as the name of the `snapshot id`.

The `<storage url>` would look like this:
~~~
s3://us-east-1@gateway.storjshare.io/photosbackup
~~~

Here is the format:
~~~
$ duplicacy init -e -storage-name <name> -repository <path> <snapshot id> <storage url>
~~~

Here is the complete command:

~~~
$ duplicacy init -e -storage-name storjphotos -repository /home/curt/Photos photosbackup s3://us-east-1@gateway.storjshare.io/photosbackup
~~~

Here is the output:
~~~
$ duplicacy init -e -storage-name storjphotos -repository /home/curt/Photos photosbackup s3://us-east-1@gateway.storjshare.io/photosbackup
Enter S3 Access Key ID:
Enter S3 Secret Access Key:
Enter storage password for s3://us-east-1@gateway.storjshare.io/photosbackup:***********
Re-enter storage password:***********
/home/curt/Photos will be backed up to s3://us-east-1@gateway.storjshare.io/photosbackup with id photosbackup

~~~
You will see the following prompts:
~~~
Enter S3 Access Key ID:
Enter S3 Secret Access Key:
Enter storage password for s3://us-east-1@gateway.storjshare.io/photosbackup:
Re-enter storage password:
/home/curt/Photos/ will be backed up to s3://us-east-1@gateway.storjshare.io/photosbackup with id photosbackup
~~~

1. Type in your `S3 Access Key ID`.
2. Type in your `S3 Secret Access Key`.
3. Type in a storage password to use.

## Saving configuration information

Duplicacy automatically creates a configuration folder named `.duplicacy` after the initialization is completed.

Duplicacy stores it's configuration information in the `.duplicacy/preferences` file.

The **duplicacy set** command is used to save your keys, passwords, etc in this configuration file so you don't have to supply them manually when running a backup or restoring files. This also allows you to automate your backups with bash scripts and to schedule them via cron jobs.

~~~
duplicacy set -storage <storage name> -key -value
~~~

For the full list of environment variables that can be set in the preferences file see the official Duplicacy [forum](https://forum.duplicacy.com/t/passwords-credentials-and-environment-variables/1094).

We'll be setting the following variable keys:

~~~
<storagename>_password
<storagename>_s3_id
<storagename>_s3_secret
~~~
The variable `key` needs to be all lowercase and with double quotes. The `value` variable needs to be in single quotes.

Here is the complete command for our example:
~~~
$ duplicacy set -storage storjphotos -key "storjphotos_password" -value 'mystoragepassword'
$ duplicacy set -storage storjphotos -key "storjphotos_s3_id" -value 'myS3IDkey'
$ duplicacy set -storage storjphotos -key "storjphotos_s3_secret" -value 'myS3secretkey'
~~~

Once this is configured, you can run the backup command without being prompted to provide your S3 credentials and storage password.

## Backing up

Now we're ready to backup our data using the **duplicacy backup** command.

~~~
duplicacy backup -storage <storagename>
~~~

Here is the complete command using our example:

~~~
$ duplicacy backup -storage storjphotos
~~~

Here is the output:

~~~
$ duplicacy backup -storage storjphotos
Repository set to /home/curt/Photos
Storage set to s3://us-east-1@gateway.storjshare.io/photosbackup
No previous backup found
Indexing /home/curt/Photos
Parsing filter file /home/curt/.duplicacy/filters
Loaded 0 include/exclude pattern(s)
Listing all chunks
Packed profile.png (133686)
Backup for /home/curt/Photos at revision 1 completed
~~~

Duplicacy assigns a revision number starting at 1 and increments it every time you run the backup.

Here is what a second backup will look like if there are no file changes.

~~~
$ duplicacy backup -storage storjphotos
Repository set to /home/curt/Photos
Storage set to s3://us-east-1@gateway.storjshare.io/photosbackup
Last backup at revision 1 found
Indexing /home/curt/Photos
Parsing filter file /home/curt/.duplicacy/filters
Loaded 0 include/exclude pattern(s)
Backup for /home/curt/Photos at revision 2 completed
~~~

## Restoring files

The **duplicacy restore** command is used to restore files from your backups.
~~~
duplicacy restore [command options] [--] [pattern] ...
~~~

The **duplicacy restore** command has multiple options, but we'll only be discussing the following:

~~~
-r <revision>				      the revision number of the snapshot (required)
-stats 					          show statistics during and after restore
-overwrite 					      overwrite existing files in the repository
-storage <storage name> 	restore from the specified storage instead of the default one
~~~
To restore the entire snapshot from the initial backup, you would run the following command:

~~~
$ duplicacy restore -storage storjphotos -stats -r 1
~~~

Here is the output:

~~~
$ duplicacy restore -storage storjphotos -stats -r 1
Repository set to /home/curt/Photos
Storage set to s3://us-east-1@gateway.storjshare.io/photosbackup
Loaded 0 include/exclude pattern(s)
Forcing in-place mode with a non-default preference path
Parsing filter file /home/curt/.duplicacy/filters
Loaded 0 include/exclude pattern(s)
Restoring /home/curt/Photos to revision 1
Downloaded chunk 1 size 133686, 131KB/s 00:00:01 100.0%
Downloaded profile.png (133686)
Restored /home/curt/Photos to revision 1
Files: 1 total, 131K bytes
Downloaded 1 file, 131K bytes, 1 chunks
Skipped 0 file, 0 bytes
Total running time: 00:00:01
~~~
This will only restore files that are missing from your original directory. It will not overwrite any existing files unless you use the `-overwrite` option. 

Please note, the `overwrite` option will only overwrite existing files that have been changed. If there are no changes between the file in snapshot and the file in the existing directory, the restore command will ignore the `overwrite` option.

If you want to only restore specific files you can specify a pattern.

To restore all files with the **.png** extension:
~~~
$ duplicacy restore -storage storjphotos -stats -r 1 *.png
~~~

To restore a file named **profile.png**:
~~~
$ duplicacy restore -storage storjphotos -stats -r 1 profile.png
~~~

## Listing files in the backups

The **duplicacy list** command is used to list files that are in the backup snapshots.
~~~
duplicacy list [command options] 
~~~

The **duplicacy list** command has multiple options, but we'll only be discussing the following:
~~~
-storage <storage name> 	retrieve snapshots from the specified storage
-r <revision>	    	      the revision number of the snapshot
-files 			              print the file list in each snapshot
~~~

To list the files from my 4th backup:
~~~
$ duplicacy list -storage storjphotos -r 4 -files
~~~

Here is the output:
~~~
$ duplicacy list -storage storjphotos -r 4 -files
Repository set to /home/curt/Photos
Storage set to s3://us-east-1@gateway.storjshare.io/photosbackup
Snapshot photosbackup revision 4 created at 2022-11-11 15:28 
Files: 3
133686 2022-10-21 14:59:23 769944b45a9f9f72061effada985d639c5fbf8fb3184a7e2c55875707f60ea80 profile.png
50282 2022-10-26 08:38:10 591bfd3d567d1e0550030aa515383298cefc3919c3ad78c5a50e442de7943fde redhat.png
  6 2022-11-11 15:27:51 a10c79f8874d13a67ce4679ea3faeaba23217febc1e446aa47812701cf9244d5 test.log
Files: 3, total size: 183974, file chunks: 3, metadata chunks: 3
~~~

## Conclusion
Duplicacy is a very powerful backup tool and this tutorial is only a brief introduction on what it has to offer.

Be sure to check out the Duplicacy [forum](https://forum.duplicacy.com/) for complete documentation and great community support.
