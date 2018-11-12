..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 https://creativecommons.org/licenses/by/3.0/legalcode

=============================================
Freezer Cinder Volumes backup using OS Bricks
=============================================

* https://blueprints.launchpad.net/freezer/+spec/cinder-osbrick

Provide efficient way to backup Cinder Volumes leveraging os_bricks

Problem description
===================

Currently Freezer provides basic features to execute Cinder volumes backup.
The current approach present significant challenges,
due mainly to the difficulty of downloading Cinder Volumes without passing
through Glance. This can be an issue for time and scalability reasons,
(i.e. volumes of few hundreds GB size, potential error probability increase,
as more services are part of the process, unailability of cinder-backup)

Use Cases
---------

* Users that want to backup cinder volumes.

* Store backed up volumes in a different storage media than Swift.
  This is important for disaster recovery purpose, as it should be
  possible to restore the volume even if the swift or other services are
  down in the original OpenStack deployment.

* OpenStack distributions deployed without cinder-backup module.

* Provide a more efficient way of executing incrementals backup.

* Avoid uploading volumes image to Glance to be processed.

* Volumes can be backed up while attached or detached (hot and cold).

- Hot backup will be provide a crach consistent backup and the data  present
  in the volumes can be accesses at all times during backup
- If Cold backup is executed, the Volume is detached first,
  then the backup is executed.


Proposed change
===============

Implement in the freezer-agent a new engine called cinder-osbrick.

The new engine cinder-osbrick execute backup and restore related
operations direclty on the Volumes, without passing through Glance API.

The freezer-agent needs to back up a single volume, all the volumes
owned by the tenant or all volumes from all tenants (admin).
Volumes backup and restore can happen in parallel (i.e. 10 Volumes
simultaneously can be backup or restore)


Technical details
-----------------

OpenStack provide the os_brick library to attach volumes:

* https://opendev.org/openstack/os-brick

It mainly provides the following features:

* Volumes discovery
* Volumes attach
* Volumes removal

Related docs:

* https://docs.openstack.org/os-brick/latest/reference/index.html
* https://docs.openstack.org/os-brick/latest/user/tutorial.html

The python client module that could be used is brick-cinderclient-ex:

* https://opendev.org/openstack/python-brick-cinderclient-ext

It is preferrable to implement the Volumes related operations from cinder
in python, rather wrapping around any possible related os-brick command.

* The freezer-scheduler and the freezer-agent needs to support,
  in the json and ini config file respectively, engine specific settings.


Backup workflow with osbrick:
-----------------------------

* freezer-agent workflow:

(Common steps)

single-vol-backup:
 - Backup any available metadata of the Volume
 - A Snapshot is execute on the volume (--force if volume is attached)
 - Snapshot is converted to Volume, in oder to be mounted using osbrick
 - The new Volume is attached using os_brick. The Volume can be attached
   using iSCSI, Local or FC according the information provided by
   os_brick about the volume.
 - The new Volume is mounted on the node where the freezer-agent is executing
 - The freezer-agent will execute a backup of the volume content, starting
   from the volume mount (i.e. volume root /)
 - Every single file in the volume is backed up.
 - If the execution is part of an incremental backup, each file/block is
   compared against the previous execution.
 - Data can be storage on each supported freezer storage backend
 - When finished, the new Volume is detached
 - Once detached, the new volume is removed

Backup of a Single Volume:
 1) The freezer-agent take the volume id as input param (either from ini
    file or json file provided to the scheduler):
 2) single-vol-backup from Common steps

Backup of all Volumes owned by a tenant:
 1) freezer-agent discover all the volumes owned by the tenant from Cinder API
 2) Iterate over each Volume
 3) single-vol-backup from Common steps

Backup of all Volumes (admin):
 1) freezer-agent get the list of all Volumes available from Cinder API
 2) Iterate over each volume
 3) single-vol-backup from Common steps

Backup of all volumes part of a Consistency Group
 1) get list of all volumes from the Consistency Group. It can be provided
    as a single element id or a list of elements comma separated:
 2) freezer-agent get the list of all Volumes available from Cinder API
 3) Iterate over each volume
 4) single-vol-backup from Common steps


Restore workflow with osbrick
-----------------------------

* freezer-agent workflow:

(Common steps)

single-vol-restore:
  - Get the original Volume metadata
  - Check if the volume id exists
  - If the same volume id exists

    + snapshot the volume
    + convert from snap to volume
    + attach the volume
    + mount the volume
    + restore the backup data in the volume filesystem
    + if meta-override option is provided, the volume metadata from backup
      is applied to the current Volume meta
  - If the volume id does not exist
    + Create a new Volume with the same metadata from backup
    + attach the volume with os-brick
    + mount the volume
    + restore the backup data in the volume filesystem
  - unmount
  - deattach the volum
  - if remove_old_vol is provided, any existing volume not matching with the
    new ones will be removed (Dangerous Option)

Restote of a single volume:
 1) The freezer-agent take the volume id as input param (either from ini
    file or json file provided to the scheduler):
 2) single-vol-restore from Common steps

Restore of all volumes owned by a tenant:
 1) freezer-agent discover all the volumes owned by the tenant from Cinder API
 2) Iterate over each volume
 3) single-vol-restore from Common steps

Restore of all volumes from all tenants (admin):
 1) freezer-agent get the list of all Volumes available from Cinder API
 2) Iterate over each volume
 3) single-vol-restore from Common steps

Restore of all volumes part of a Consistency Group
 1) get list of all volumes from the Consistency Group. It can be provided as
    a single element id or a list of elements comma separated:
 2) Iterate over each volume
 3) single-vol-restore from Common steps

Data model impact
-----------------
* new engine in the db
* DB model for single, all tenant, tenant owned volumes and consistency groups

New Options to be added:
------------------------
* engine-os-brick
* recreate-vol-on-error
* meta-override
* consistency groups [id]
* all_tenants
* all_tenant_volumes
* single_volume_id
* remove_old_vol

Alternatives
------------


Impacts
-------
* freezer-agent
* freezer-api
* freezer-web-ui

REST API impact
-----------------------

* API needs to support this new engine

Security impact
---------------------

None

Notifications impact
---------------------------

TBD.

Other end user impact
------------------------------

None. TBD.

Performance Impact
------------------

None.

Other deployer impact
------------------------------


Developer impact
------------------------


Implementation
==============

Assignee(s)
-----------------

Primary assignee:

Other contributors:
  daemontool

Work Items
----------


Dependencies
============


Testing
=======

TBD.

Documentation Impact
====================


* Freezer API installation doc
* Freezer agent docu
* Freezer web ui doc

References
==========

.. https://etherpad.openstack.org/p/freezer_cinder-os-brick

