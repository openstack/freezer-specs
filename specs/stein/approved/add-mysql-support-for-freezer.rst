..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=============================
Add mysql support for freezer
=============================

* https://storyboard.openstack.org/#!/story/2004132

Provide more db options for users to store the management data.

Problem description
===================

Currently, freezer can only support elasticsearch as db management engine,
so we should increase support for mysql .

Use Cases
---------

* Users that want to backup management data to mysql database.

* OpenStack distributions deployed without elasticsearch.

Proposed change
===============

Implement in the freezer-api a new db engine mysql.

With specifying the backend = sqlalchemy and driver = sqlalchemy in storage
section of conf file , the management data will be stored on mysql db server

Alternatives
------------

None

Technical details
-----------------

Related docs:
-----------------

None

Data model impact
-----------------

* Add clients table::

    clients = Table(
        'clients', meta,
        Column('created_at', DateTime(timezone=False)),
        Column('updated_at', DateTime(timezone=False)),
        Column('deleted_at', DateTime),
        Column('deleted', Boolean),
        Column('user_id', String(36), nullable=False),
        Column('id', String(255), primary_key=True, nullable=False),
        Column('project_id', String(36), nullable=False),
        Column('config_id', String(255), nullable=False),
        Column('hostname', String(255), nullable=False),
        Column('description', String(255)),
        Column('uuid', String(36), nullable=False),
        mysql_engine='InnoDB',
        mysql_charset='utf8'
    )

* Add sessions table::

    sessions = Table(
        'sessions', meta,
        Column('created_at', DateTime(timezone=False)),
        Column('updated_at', DateTime(timezone=False)),
        Column('deleted_at', DateTime),
        Column('deleted', Boolean),
        Column('id', String(36), primary_key=True, nullable=False),
        Column('session_tag', Integer, default=0),
        Column('description', String(255)),
        Column('hold_off', Integer, default=30),
        Column('scheduling', Text),
        Column('job', Text),
        Column('project_id', String(36), nullable=False),
        Column('user_id', String(36), nullable=False),
        Column('time_start', Integer),
        Column('time_end', Integer),
        Column('time_started', Integer),
        Column('time_ended', Integer),
        Column('status', String(255)),
        Column('result', String(255)),
        mysql_engine='InnoDB',
        mysql_charset='utf8'
    )

* Add jobs table::

    jobs = Table(
        'jobs', meta,
        Column('created_at', DateTime(timezone=False)),
        Column('updated_at', DateTime(timezone=False)),
        Column('deleted_at', DateTime),
        Column('deleted', Boolean),
        Column('id', String(36), primary_key=True, nullable=False),
        Column('project_id', String(36), nullable=False),
        Column('user_id', String(36), nullable=False),
        Column('scheduling', Text),
        Column('client_id', String(255), nullable=False),
        Column('session_id', String(36), default=''),
        Column('session_tag', Integer, default=0),
        Column('description', String(255)),
        Column('job_actions', Text),
        mysql_engine='InnoDB',
        mysql_charset='utf8'
    )

* Add actions table::

    actions = Table(
        'actions', meta,
        Column('created_at', DateTime(timezone=False)),
        Column('updated_at', DateTime(timezone=False)),
        Column('deleted_at', DateTime),
        Column('deleted', Boolean),
        Column('id', String(36), primary_key=True, nullable=False),
        Column('action', String(255), nullable=False),
        Column('project_id', String(36), nullable=False),
        Column('user_id', String(36), nullable=False),
        Column('mode', String(255)),
        Column('src_file', String(255)),
        Column('backup_name', String(255)),
        Column('container', String(255)),
        Column('timeout', Integer),
        Column('priority', Integer),
        Column('max_retries_interval', Integer, default=6),
        Column('max_retries', Integer, default=5),
        Column('mandatory', Boolean, default=False),
        Column('log_file', String(255)),
        Column('backup_metadata', Text),
        mysql_engine='InnoDB',
        mysql_charset='utf8'
    )


* Add backups table::

    backups = Table(
        'backups', meta,
        Column('created_at', DateTime(timezone=False)),
        Column('updated_at', DateTime(timezone=False)),
        Column('deleted_at', DateTime),
        Column('deleted', Boolean),
        Column('id', String(36), primary_key=True, nullable=False),
        Column('client_id', String(255), nullable=False),
        Column('job_id', String(36), nullable=False),
        Column('project_id', String(36), nullable=False),
        Column('user_id', String(64), nullable=False),
        Column('user_name', String(64)),
        Column('backup_metadata', Text),
        mysql_engine='InnoDB',
        mysql_charset='utf8'
    )


REST API impact
---------------
None

Security impact
---------------
None

Notifications impact
--------------------
Some message log will be added.

Other end user impact
---------------------
None

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
  gecong <ge.cong@zte.com.cn>
  gengchc2 <geng.changcai2@zte.com.cn>

Work Items
----------

* Implementing the db engine driver (mysql)

* Implementing the mysql api

* Implementing the mysql tables  creation and upgrade

Dependencies
============
None

Testing
=======
add unit test

Documentation Impact
====================
Some instructions should be added .

References
==========
None

History
=======
None
