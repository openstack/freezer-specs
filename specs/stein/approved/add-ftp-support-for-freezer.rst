..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=========================================
Add FTP,SFTP and FTPS support for freezer
=========================================

* https://storyboard.openstack.org/#!/story/2004332

Provide FTPS, FTP, SFTP options for users to store backup data.

Problem description
===================

Currently, freezer can only support local, ssh(support ssh_key_path), swift, s3
 as backup data storage, so we should increase support for FTP, FTPS(support
 password).

Use Cases
---------

* Users that want to backup data to FTP, FTPS, SFTP server.

Proposed change
===============

Implement in the freezer new storages.

With specifying the storage as FTP, FTPS or SFTP,
the backup data will be stored on FTP, FTPS or SFTP server.

For all these new added storage type, no Database models changes
will happen. As matter of fact, we use python FTP class to implement
details of FTP Client-server communication and authentication.

For FTP storage, we will use class ftplib.FTP. The class of ftplib.FTP_TLS
will be used to implement FTPS storage. And for SFTP storage, the python
module paramiko is a suitable choice.

Alternatives
------------

None

Technical details
-----------------

Related docs:
-------------
None

Data model impact
-----------------
None

REST API impact
---------------
* Specifying the storage as ftps, the request Example
  in creating job as follows::

    {
        "description": "Test-0001",
        "job_schedule": {
            "schedule_interval": "5 minutes",
            "status": "scheduled",
            "event": "start"
        },
        "job_actions": [
            {
                "max_retries": 5,
                "max_retries_interval": 6,
                "freezer_action": {
                    "backup_name": "test0001_backup",
                    "storage": "ftps",
                    "ftp_host": "192.177.10.44",
                    "ftp_username": "ftpstest",
                    "ftp_password": "pass_test",
                    "ftp_port": 21,
                    "container": "/home/ftpstest",
                    "no_incremental": true,
                    "path_to_backup": "/etc/",
                    "log_file": "/home/ftp_data/job0001.log",
                    "snapshot": true,
                    "action": "backup",
                    "remove_older_than": 10,
                }
            }
        ]
    }

* Specifying the storage as ftp, the request Example
  in creating job as follows::

    {
        "description": "Test-0001",
        "job_schedule": {
            "schedule_interval": "5 minutes",
            "status": "scheduled",
            "event": "start"
        },
        "job_actions": [
            {
                "max_retries": 5,
                "max_retries_interval": 6,
                "freezer_action": {
                    "backup_name": "test0002_backup",
                    "storage": "ftp",
                    "ftp_host": "192.177.10.44",
                    "ftp_username": "ftptest",
                    "ftp_password": "pass_test",
                    "ftp_port": 21,
                    "container": "/home/ftptest",
                    "no_incremental": true,
                    "path_to_backup": "/etc/",
                    "log_file": "/home/ftp_data/job0002.log",
                    "snapshot": true,
                    "action": "backup",
                    "remove_older_than": 10,
                }
            }
        ]
    }

* Specifying the storage as ftps, the request Example
  in creating job as follows::

    {
        "description": "Test-0001",
        "job_schedule": {
            "schedule_interval": "5 minutes",
            "status": "scheduled",
            "event": "start"
        },
        "job_actions": [
            {
                "max_retries": 5,
                "max_retries_interval": 6,
                "freezer_action": {
                    "backup_name": "test0003_backup",
                    "storage": "sftp",
                    "ftp_host": "192.177.10.44",
                    "ftp_username": "ftptest",
                    "ftp_password": "pass_test",
                    "ftp_port": 22,
                    "container": "/home/sftptest",
                    "no_incremental": true,
                    "path_to_backup": "/etc/",
                    "log_file": "/home/ftp_data/job0003.log",
                    "snapshot": true,
                    "action": "backup",
                    "remove_older_than": 10,
                }
            }
        ]
    }

Security impact
---------------
None

Notifications impact
--------------------
Some message log will be added.

Other end user impact
---------------------
* Add these parammeters as follows in freezer-agent::
  ftp_host
  ftp_username
  ftp_password
  ftp_port

* usecase in freezer-agent using ftp as storage::
   sudo freezer-agent --path-to-backup /data/dir/to/backup
   --container /remote-machine-path/ --backup-name my-backup-name
   --storage ftp --ftp-username ftpuser --ftp-password passwd_test
   --ftp-host 8.8.8.8 --ftp-port 82

* usecase in freezer-agent using ftps as storage::
   sudo freezer-agent --path-to-backup /data/dir/to/backup
   --container /remote-machine-path/ --backup-name my-backup-name
   --storage ftps --ftp-username ftpuser --ftp-password passwd_test
   --ftp-host 8.8.8.8 --ftp-port 82

* usecase in freezer-agent using sftp as storage::
   sudo freezer-agent --path-to-backup /data/dir/to/backup
   --container /remote-machine-path/ --backup-name my-backup-name
   --storage sftp --ftp-username ftpuser --ftp-password passwd_test
   --ftp-host 8.8.8.8 --ftp-port 82

Performance Impact
------------------
None

Other deployer impact
---------------------

Developer impact
----------------
None

Implementation
==============
Assignee(s)
-----------

Primary assignee:
  gengchc2<geng.changcai2@zte.com.cn>
  gecong <ge.cong@zte.com.cn>

Work Items
----------

* Implementing ftp storage in freezer

* Implementing ftps storage in freezer

* Implementing sftp storage in freezer

* Adding these parameters ftp_host, ftp_username, ftp_password, ftp_port in
freezer-agent

Dependencies
============
None

Testing
=======
add unit test

Documentation Impact
====================
Some instructions should be added.

References
==========
None

History
=======
None
