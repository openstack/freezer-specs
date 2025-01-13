..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=======================================================
Backup all projects using centralized freezer-scheduler
=======================================================

https://blueprints.launchpad.net/freezer/+spec/centralized-scheduler

Add ability to schedule backups of Cinder volumes using centralized
instance(s) of freezer-scheduler.

Problem description
===================

Currently, Freezer requires an instance of the freezer-scheduler
to be deployed per project.

For OpenStack deployments, which perform cinder-native backups only,
this approach can result in redundant processes and higher resource
consumption. A more efficient solution would involve enabling centralized
instance(s) of the freezer-scheduler to handle backup operations across
multiple projects within an OpenStack deployment.

Use Cases
---------

* As a cloud administrator, I want to manage backups for multiple projects
  using a single instance of the freezer-scheduler, reducing operational
  overhead and simplifying infrastructure management.

* As a cloud user, I want to request backups without concern for the
  underlying scheduler architecture.

Proposed change
===============

Implement a new mode called "service mode" in the freezer-scheduler.
The freezer-scheduler, running in service mode, will create backups for
different users and projects.

Add ability to retrieve a list of jobs for all users in the freezer-api.
When processing a request to obtain the job list for all users, verify that
the requester has a role=service or role=admin.

Modify the freezer-scheduler to retrieve a list of jobs for all users.
Use trusts to allow the freezer-scheduler to access user resources
and store backups on behalf of users.

Add corresponding configuration options and restrict available backup modes
on the freezer-scheduler running in service mode to allow only creation of
cinder-native backups. The absence of such restrictions could result in
unexpected workloads and security issues (for example, creating a filesystem
backup from the system where the freezer-scheduler is running).

Implement a verification mechanism to confirm that the client for whom the
job is being created supports the backup mode specified by the user.

Alternatives
------------

An alternative way is to deploy a freezer-scheduler per project.
This method is more resource consuming for cinder-native backups.

Data model impact
-----------------

Add new fields that represent freezer-scheduler capabilities to
``clients`` table:

* ``supported_actions`` - list of supported backup actions

* ``supported_modes`` - list of supported backup modes

* ``supported_storages`` - list of supported storage types

* ``supported_engines`` - list of supported backup engines

Add new table ``user_credentials`` to store trusts:

* ``id`` - String(36) primary key

* ``decrypt_method`` - string(64)

* ``trust_id`` - string(255)

* ``trustor_user_id`` - string(64)

* ``job_id`` - String(36) foreign key

REST API impact
---------------

* POST ``/v2/{project_id}/clients`` - add new fields to the request
  body: ``supported_actions``, ``supported_modes``, ``supported_storages``,
  ``supported_engines``.

* GET ``/v2/{project_id}/clients`` - add new fields to the client:
  ``supported_actions``, ``supported_modes``, ``supported_storages``,
  ``supported_engines``.

* GET ``/v2/{project_id}/clients/{client_id}`` - add new fields to the
  client: ``supported_actions``, ``supported_modes``, ``supported_storages``,
  ``supported_engines``.

* GET ``/v2/{project_id}/jobs``` add a new option ``all_projects`` and new
  fields ``trust_id``, ``trustor_user_id``. Return jobs for all projects when
  option ``all_projects`` is set to ``True``.

* GET ``/v2/{project_id}/jobs/{job_id}`` - add new fields ``trust_id``,
  ``trustor_user_id``

Security impact
---------------

The freezer-scheduler will have access to all user resources using trusts.

Notifications impact
--------------------

None

Other end user impact
---------------------

None

Performance Impact
------------------

None

Other deployer impact
---------------------

None

Developer impact
----------------

None

Upgrade impact
--------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
    waerinfu (Volodymyr Mevsha)

Feature Liaison
---------------

Feature liaison:
  noonedeadpunk (Dmitriy Rabotyagov)

Work Items
----------

* freezer-scheduler: add ability to restrict backup modes, actions, storages,
  engines

* freezer-api: implement new fields in clients API

* freezer-scheduler: send supported actions, modes, storages, engines to
  freezer-api on registration

* freezer-api: check supported actions, modes, storages, engines when
  assigning a job to a client

* freezer-api: implement ``all_projects`` option in jobs API

* freezer-api: implement trusts including jobs API

* freezer-scheduler: implement service mode

Dependencies
============

None

Testing
=======

TBD

Documentation Impact
====================


References
==========

