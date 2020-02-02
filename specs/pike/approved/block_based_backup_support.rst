..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==================================
Block based backup support (rsync)
==================================

https://blueprints.launchpad.net/freezer/+spec/rsync

Taking advantage of the rsync to provide a possibility to create
space/bandwidth efficient backups.

Problem description
===================

Currently Freezer checks only ctime and mtime inode information
to verify if files are changed or not (tar functionality). While
this approach gives speed (time efficient), it is not bandwidth
and storage efficient. Freezer needs to support both rsync and tar
approach to execute incremental backups and restore.

Since Freezer will provide two options for incremental backups, it
would be more convenient to choose the best approach to backup data
in accordance with each particular case (more speed or storage/bandwidth
efficient).

Use Cases
---------

* For developers, this change will not create negative impacts because
  this code will be gracefully bundled in Freezer engine API and
  will not cause any major changes in Freezer architecture.

* For Deployers there is no need to install any additional components,
  Freezer will use it's own implementation of rsync algorithm
  (written in Python).

* For End User it would be less difficult to select more efficient
  option for create backups based on dataset (e.g. few big files or a lot of
  small files) for backup and speed/storage/bandwidth requirements,
  since Freezer would support both rsync and tar approaches.

Proposed change
===============

Implementing the new engine classes for rsync (as well as for tar).
Providing new engine (-e) choice in config.

For this type of backup will be created following metadata structure::

  files_meta = {
            'files': {},
            'directories': {},
            'meta': {
                'broken_links_tot': '',
                'total_files': '',
                'total_directories': '',
                'backup_size_on_disk': '',
                'backup_size_uncompressed': '',
                'backup_size_compressed': '',
                'platform': sys.platform
            },
            'abs_backup_path': os.getcwd(),
            'broken_links': [],
            'rsync_struct_ver': RSYNC_DATA_STRUCT_VERSION,
            'rsync_block_size': RSYNC_BLOCK_SIZE}


  file_meta = {'inode': {
                      'inumber': os_stat.st_ino,
                      'nlink': os_stat.st_nlink,
                      'mode': file_mode,
                      'uid': os_stat.st_uid,
                      'gid': os_stat.st_gid,
                      'size': os_stat.st_size,
                      'devmajor': os.major(dev),
                      'devminor': os.minor(dev),
                      'mtime': mtime,
                      'ctime': ctime,
                      'uname': uname,
                      'gname': gname,
                      'ftype': file_type,
                      'lname': lname,
                      'rsync_block_size': rsync_block_size,
                      'file_status: status
                      }
            }

Current version of implementation you always can find here [1]_.

Alternatives
------------

Because of the flexibility, speed, and scriptability of rsync, it has
become a standard Linux utility, included in all popular Linux distributions.
It has been ported to Windows (via Cygwin, Grsync, or SFU), FreeBSD, NetBSD,
OpenBSD, and Mac OS. De facto, rsync is the default fallback for most data
transfers. It has a clear algorithm written for 20 years ago and different
variations (e.g. acrosync, zsync, etc). librsync is used by Dropbox.

Using other alternative (like bbcp or lftp) would not be more effective
or portable solution.

Data model impact
-----------------

Changes in data model has already described in oslo.db migration document.
Actions entity should contain 'engine' field for performing appropriate action
using particular type of engine (tar, rsync or openstack).

From new relational database schema:

Actions
    action_id (uuid) [p_key]
    resource (varchar)
    type (varchar)
    name (varchar)
    application (varchar)
    engine (varchar)    <-- Require this
    snapshot (varchar)
    storage (varchar)
    global_options (JSON)
    application_options (JSON)
    storage_options (JSON)
    snapshot_options (JSON)
    engine_options (JSON)


REST API impact
---------------

None.

Security impact
---------------

None.

Notifications impact
--------------------

There are no special logs will be added, just some info messages about
start/stop backup process, backup metrics, etc.

Other end user impact
---------------------

* There are no additional changes to python-freezerclient CLI. To choice
  appropriate engine for action, end user should specify 'engine' field
  in provided JSON configuration in case of creating or updating action.

* freezer-web-ui should provide additional 'engine' field in 'Action
  Configuration' window. It has to be drop-down list with values 'tar',
  'rsync' or 'openstack'.

Performance Impact
------------------

None.

Other deployer impact
---------------------

Will be added new choice to freezer-agent -e (engine) option - 'rsync'.

Developer impact
----------------

None.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Ruslan Aliev (raliev) <raliev@mirantis.com>

Other contributors:
  Fausto Marzi (daemontool) <fausto.marzi@ericsson.com>

Work Items
----------

* implementing the new engine (rsync)

* bundling this engine to freezer code (API calls) and mechanism
  for using this engine ('-e rsync' option)

* implementing the new database schema for actions (oslo.db migration)

* updating freezer-web-ui 'Action Configuration' window

* updating documentation

Dependencies
============

* This spec depends on Freezer oslo.db migration [2]_.

* Pluggable engines described here [3]_.

* There are no additional library dependencies.

Testing
=======

There is a question - do we actually need separate tempest test
for this change or we can be satisfied with existing one?

Documentation Impact
====================

* freezer README doc

* freezer-api README doc

* freezer-web-ui README doc

References
==========

.. [1] https://review.openstack.org/#/c/409796/
.. [2] https://etherpad.openstack.org/p/freezer_mysql_migration
.. [3] https://etherpad.openstack.org/p/freezer_new_archi
