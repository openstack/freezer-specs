..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=========================
S3 storage driver support
=========================

* https://blueprints.launchpad.net/freezer/+spec/s3-driver-support

Provide more storage media options for users to store the backup data.

Problem description
===================

Currently, freezer can only store backup data to swift compatible object
storage (except local and ssh), so we should increase support for other storage
driver. S3 compatible object storage is a valid choice, which is used by many
individuals and companies in the public or private clouds. With S3 driver,
freezer can store backup data in S3 compatible object storage, such as AWS S3
or ceph S3 interface.

Use Cases
---------

* Users that want to backup data to S3 compatible object storage.

* OpenStack distributions deployed without swift module.

Proposed change
===============

Implement in the freezer-agent a new storage type called 's3'.

With specifying the 's3' storage type, the authentication parameters and the
S3 endpoint, backup data will be stored on S3 compatible storage after the
backup finished.

Alternatives
------------

None

Technical details
-----------------

Related docs:

Amazon S3 REST API Introduction
* http://docs.aws.amazon.com/AmazonS3/latest/API/Welcome.html

The python client module that could be used is boto3
* https://github.com/boto/boto3

Data model impact
-----------------

None

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

* A new storage type called 's3' will be added.
* When specify 's3' type, 'access-key', 'secret-key' and 'endpoint' should be
required.

Developer impact
----------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Pengju Jiao <jiaopengju@cmss.chinamobile.com>

Work Items
----------

* Implementing the new storage driver (S3)

* Bundling the storage driver to freezer-agent


Dependencies
============

None


Testing
=======

None

Documentation Impact
====================

Some instructions should be added to tell users how to use S3 storage driver.


References
==========

None


History
=======

None