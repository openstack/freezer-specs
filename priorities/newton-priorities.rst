.. _newton-priorities:

=========================
Newton Project Priorities
=========================

List of themes (in the form of use cases) the freezer development team will
prioritize in Newton.

+-------------------------------------------+-----------------------+
| Priority                                  | Primary Contacts      |
+===========================================+=======================+
| `Hardening and testing`                   | `Jonas Pfannschmidt`_ |
+-------------------------------------------+-----------------------+
| `Agent refactoring`_                      | `Saad zaher`_         |
+-------------------------------------------+-----------------------+
| `Documentation improvement`_              | `Guillermo Garcia`_   |
+-------------------------------------------+-----------------------+
| `Disaster recovery`_                      | `Fabrizio Fresco`_    |
+-------------------------------------------+-----------------------+

.. _Jonas Pfannschmidt: https://launchpad.net/~jonas-pfannschmidt
.. _Saad zaher: https://launchpad.net/~szaher
.. _Fabrizio Fresco: https://launchpad.net/~felipe
.. _Guillermo Garcia: https://launchpad.net/~sirmemogarcia


Hardening and testing
--------

In Newton, we want to increase as much as possible the test coverage for
Freezer.
This include better and more unit tests and tempest tests.

Agent refactoring
---------

In Newton, we want to make our freezer-agent pluggable for four of its core
internal parts:

* Pluggable storage: In order to support more storage backend
* Pluggable snapshot: In order to support more snapshoting technique (like xfs,
  brtfs, ...)
* Pluggable application: Through a system of hooks, we want to facilitate the
  support of new application aware backups.
* Pluggable engine: In order to support other ways of processing data.

Disaster recovery
----------------

In Newton we will kick off freezer-dr in order to provide tooling to support
disaster recovery cases where backup/restore is not able to provide a good enough RTO.

Documentation improvement
------------------

In Newton, we will refactor our documentation. This includes mooving it away
from the READMEs, refactoring and updating the content as well as improving the
developper documentation.
