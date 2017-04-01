..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Freezer puppet manifests spec
==========================================

https://blueprints.launchpad.net/freezer/+spec/freezer-puppet-manifests

Problem description
===================

As there is no any automation tools for Freezer it is proposed to develop a
set of puppet manifests for auto install the following Freezer components:

* Freezer (freezer-agent and scheduler);
* Freezer API;
* Freezer Horizon Web UI;

Optionally we can consider possible to add set of manifests for Freezer
Disaster Recovery.

Use Cases
---------

All implemented manifest should follow principles of idempotence.

This bundle should make Freezer deployment pretty convenient and simple for
end-users and shouldn't require additional pieces of knowledge regarding
Freezer infrastructure during the first installation.

Also, these manifests should significantly simplify maintenance of Freezer
modules.


Proposed change
===============

* All Freezer puppet manifest should be located in separate repo:
  https://github.com/openstack/puppet-freezer
* Puppet community requires having packages for various operating systems.
  Nevertheless, for a while can be used package puppet module

 * RPM packages should be discussed in #rdo IRC channel on Freenode and then
   added in RDO trunk according to the RDO `guaidliness <https://www.rdoproject.org/documentation/rdo-packaging/#how-to-add-a-new-package-to-rdo-trunk>`_
 * Debian packaging of OpenStack should be discussed in #openstack-pkg IRC
   channel on Freenode

* at the very beginning puppet manifests should be implemented for Freezer,
  Freezer-API and python-freezerclient components and only after that for
  Freezer-WebUI and Freezer-DR

Alternatives
------------

This functionality can be implemented in various ways like ansible playbooks,
shell scripts salt formulas, however, it was decided to start from puppet
manifests.

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

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  vnogin (Vitaliy Nogin)

Other contributors:
  <None>

Work Items
----------

The following task should be done according to this spec:

* Initialization of puppet-freezer repo creation;
* Create RPM packages;
* Created DEB packages;
* Implement puppet manifests for:

 * handling required system packages;
 * Freezer components;
 * Freezer-API components;
 * python-freezerclient components;
 * Freezer-WebUI components;
 * Freezer-DR components;

* Create relevant guidelines;
* Develop unit tests

Dependencies
============

None

Testing
=======

None

Documentation Impact
====================

Set of runbook and additional documentation should be implemented with full
description how to use Freezer puppet manifests during deployment and
maintenance procedures.

References
==========

Weekly Meeting Logs[1]

[1] http://eavesdrop.openstack.org/meetings/freezer/2017/freezer.2017-03-02-14.00.log.txt


History
=======

.. list-table:: Revisions
   :header-rows: 1

   * - Release Name
     - Description
   * - Pike
     - Introduced
