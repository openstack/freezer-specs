..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=================================
Relational DB Schema with OSLO.DB
=================================

https://blueprints.launchpad.net/freezer/+spec/oslo.db

Taking advantage of the oslo.db library to have a more uniform database
backend architecture to other OpenStack projects.

Problem description
===================

Currently Freezer uses Elastic Search (ES) as a database backend, which
is a NoSQL database specialized for ranked query results. Elastic Search
adds additional complexity to an OpenStack system. Most of the
components use a relational database management system (DBMS like MySQL or
PostgreSQL) which are more common. It is more familiar how to
maintain, troubleshoot and develop on top of relational databases.

Since Freezer related data turned out to be relational, it would be more
convenient to use it trough the oslo.db pattern library. Using it, the
database mapping would be more uniform to other OpenStack projects.
It would be less challenging for new developers to contribute.

Use Cases
---------

* For new developers, already familiar with OpenStack, it should be less
  challenging to get familiar with the backend code, since most of the
  OpenStack projects use relational database backend trough the oslo.db
  pattern library.

* For Deployers there would be no longer needed to set up a special
  DBMS just for Freezer, since it could share the relational DBMS used
  by the other (core) OpenStack projects, still well isolated in it's
  own database.

* For End User it would be less difficult to maintain, since Freezer
  would not add additional complexity with a less common component,
  instead it can take advantage of the DBMS that is already deployed for
  OpenStack.

Proposed change
===============

Implementing the entities using oslo.db and SQLAlchemy base classes.
And expose the new entities trough the REST API.

Alternatives
------------

Oslo.db with SQLAlchemy is the de facto standard for OpenStack projects
to implement database backends with relational DBMS. It provides high
level ORM mapping and abstracts the different database backends.
Therefore we gain compability with multiple relational DBMS just like
any other OpenStack component using oslo.db.

Using other alternative would either not be more uniform to other
OpenStack project tooling, either we would have to implement low level,
directly to a specific database driver (just like now with ES).

Data model impact
-----------------

Changes which require modifications to the data model often have a wider impact
on the system.  The community often has strong opinions on how the data model
should be evolved, from both a functional and performance perspective. It is
therefore important to capture and gain agreement as early as possible on any
proposed changes to the data model.

Questions which need to be addressed by this section include:

* What new data objects and/or database schema changes is this going to
  require?

* What database migrations will accompany this change.

* How will the initial set of new data objects be generated, for example if you
  need to take into account existing backups/jobs/... , or modify other
  existing data describe how that will work.


As there will be a brand new relational database schema [MIGR1]_:


Clients

    id (varchar) [p_key]

    project_id (uuid)

    config_id (varchar)

    description (varchar)

    uuid (uuid)



Actions

    id (uuid) [p_key]

    action (varchar)

    project_id (uuid)

    mode (varchar)

    src_file (varchar)

    backup_name (varchar)

    container (varchar(

    restore_abs_path (varchar)



Action_reports

    id (uuid) [p_key]

    action_id (uuid) [f_key]

    action_attachment_id (uuid) [f_key]

    project_id (uuid)

    result (varchar)

    time_elapsed (varchar)

    metadata (JSON)

    report_date (timestamp)

    log (blob) < only on failure



Jobs

    id (uuid) [p_key]

    project_id (uuid)

    scheduling (JSON)

    description (varchar)



Action_attachments

    id (uuid) [p_key]

    action_id (uuid) [f_key]

    job_id (uuid) [f_key]

    project_id (uuid)

    priority (int)

    retries (int)

    retry_interval (int)

    mandatory (bool)



Sessions

    id (uuid) [p_key]

    project_id (uuid)

    scheduling (JSON)

    policy (varchar)



Job_attachments

    id (uuid) [p_key]

    client_id (varchar) [f_key]

    job_id (uuid) [f_key]

    session_id (uuid) [f_key]

    project_id (uuid)

    event (varchar)

    status (varchar)

    current_pid (int)


REST API impact
---------------

There should be a new v2 API implemented. TBD.

Security impact
---------------

None

Notifications impact
--------------------

TBD.

Other end user impact
---------------------

None. TBD.

Performance Impact
------------------

None.

Other deployer impact
---------------------

* The Elastic Search configurations should be replaced with oslo.db
  configurations

* When updating from a previous version there must be a data migration
  from ES to oslo.db (this will be addressed by a nother spec - TBD).

Developer impact
----------------

There will be no longer needed to deploy ES.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  neilus

Other contributors:
  daemontool

Work Items
----------

Work items or tasks -- break the feature up into the things that need to be
done to implement it. Those parts might end up being done by different people,
but we're mostly trying to understand the timeline for implementation.

* implementing the database models

* create adapter for API v1(?) and v2

* implementing the CRUD API

* updating the devStack plugin

* updating documentation

Dependencies
============

* Implementing the database migration script (TBD), which migrates data
  from ES to oslo.db backend DB.

* We will be using oslo.db library and SQLAlchemy for iplementation.

Testing
=======

TBD.

Documentation Impact
====================

TBD.

* Freezer API installation doc

References
==========

.. [MIGR1] https://etherpad.openstack.org/p/freezer_mysql_migration

.. https://etherpad.openstack.org/p/freezer_db_switch

