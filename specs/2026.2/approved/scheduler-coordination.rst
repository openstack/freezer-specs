..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=====================================================
Freezer Scheduler Coordination and Clustering
=====================================================

https://blueprints.launchpad.net/freezer/+spec/scheduler-coordination

Allow multiple freezer-scheduler instances to run as a cluster and
share jobs among themselves instead of competing for them.

Problem description
===================

Currently, freezer does not support coordination. Each
freezer-scheduler registers itself in freezer-api as a client; its
client_id defaults to the machine's hostname but can be set in
config. Jobs are bound to one client_id and only that scheduler
runs them.

This is a problem because:

* If the scheduler dies, its jobs stop running until it is restarted —
  there is no HA.
* Two schedulers configured with the same client_id both match the
  same jobs and race to execute them — duplicate runs.

Use Cases
---------

* As a cloud administrator, I want to run multiple schedulers so jobs
  keep running when one dies.
* As a cloud administrator, I want to spread jobs across a pool of
  schedulers instead of pinning each job to one host.
* As a user, I want to make sure that my backups are created correctly
  even if the scheduler dies, and are not duplicated.

Proposed change
===============

Add coordination to freezer-scheduler while leaving freezer-api and its
data model untouched. The existing client_id is reused as the
cluster identity: operators start N schedulers all configured with the
same client_id (already supported in the scheduler config, see
``freezer/scheduler/arguments.py``). To freezer-api these schedulers
look like a single client, so they all poll the same set of jobs with
no new field and no change to the jobs filter.

All new logic lives in the scheduler. The members of a cluster are
distinguished from one another only within the coordination layer, not
in the API: each joins tooz under a unique member id, while sharing the
same group.

Coordination uses the `tooz`_ library:

1. On startup each scheduler joins a tooz group **named after its
   client_id** (the shared cluster identity), with a **unique
   member id**, and runs a heartbeat. The member id must be unique
   within the group — tooz rejects a duplicate join, and two members
   sharing an id would collapse to a single node on the hash ring. It
   is therefore built as ``<hostname>-<per-process token>`` (the token
   is the PID or a short random value). The hostname is included only
   for readability in ``coordinator.get_members()`` output and logs.
   Uniqueness across restarts is not required — a restarted scheduler's
   previous member times out and is dropped, and the fresh instance
   rejoins. tooz notifies members when peers join or leave (a peer that
   stops heartbeating is dropped after the configured timeout).
2. Each member builds a tooz ``HashRing`` from the current group
   members. To decide who runs a job, every member computes
   ``ring[job_id]`` — because the ring is identical across members,
   they all agree on the owner with no messaging.
3. Before spawning freezer-agent, the owning member takes a tooz
   distributed lock on the job id. This prevents duplicate execution
   during the brief moment when membership is changing. If a member
   dies holding the lock, the lock is released automatically when its
   tooz heartbeat expires, so a surviving member can pick the job up.

Client registration is unchanged: every member registers/updates the
same client record keyed by the shared client_id. Because that record
no longer maps 1:1 to a host, its per-host fields (hostname, etc.) are
written last-writer-wins by whichever member registered most recently.
This is cosmetic and does not affect job execution. From freezer-api's
point of view there is one client; from the coordination layer's point
of view the cluster is the tooz group client_id and its live members
are enumerable via ``coordinator.get_members()``.

If a scheduler dies mid-backup, its freezer-agent dies with it and
that run is lost. The job's new owner picks it up on the next
scheduled tick — interrupted backups are not resumed in place.
Recovery time after a member dies is approximately the tooz session
timeout plus the poll interval.

If the tooz backend itself becomes unreachable, the scheduler fails
safe: it cannot determine ownership or hold a lock, so it skips
executing jobs until coordination is restored, rather than falling
back to running everything (which would cause duplicate backups across
members). A backup that is delayed is preferable to duplicate or
racing backups, so backend availability gates execution in clustered
mode.

Coordination is opt-in. Without a coordination backend configured, the
scheduler runs standalone and does not coordinate, so schedulers
sharing a client_id will produce duplicate runs (as today).


.. _tooz: https://docs.openstack.org/tooz/latest/

Alternatives
------------

* Dedicated ``cluster`` field on the API and DB: add a nullable
  ``cluster`` column to the ``clients`` and ``jobs`` tables and expose
  it through the clients/jobs REST endpoints, so a cluster is a
  first-class, queryable object separate from client_id. Rejected
  for the initial implementation: it requires a schema migration across
  both storage drivers plus coordinated API changes, for no functional
  gain over reusing client_id. Its only advantage is observability
  — the API could then report which individual scheduler ran a job.
  With the client_id approach that information lives only in
  scheduler logs and tooz, not in the API. If per-worker visibility is
  needed later, this can be added as a follow-up without breaking the
  coordination design.

* Leader election: used instead of ``HashRing`` mechanism. The leader mechanism
  is more complex and requires additional messaging channel between
  schedulers.

* Message bus (oslo.messaging): distribute via a broker like cinder.
  Out of scope: large architectural change (broker dependency,
  affects api/scheduler/agent). Worth a separate spec if push dispatch
  or notifications are wanted.

* freezer-api as a central brain: job dispatch and scheduler
  management performed by freezer-api itself. This is a big change,
  not a standard OpenStack approach where the API is a thin layer,
  and there is no gain compared to the hash ring.

* jobboard from taskflow. A similar mechanism is implemented in
  Octavia. This avoids N api polls from each scheduler, but adds
  complexity (leader election for writing to the jobboard or idempotency).

Data model impact
-----------------

None. The cluster identity is the existing client_id; no new
columns and no migration are introduced. Multiple schedulers sharing a
client_id already map to a single ``clients`` record, and jobs
remain keyed by client_id as they are today.

REST API impact
---------------

None. Schedulers continue to register and poll using the existing
client_id on the current clients and jobs endpoints. Because the
cluster is entirely a scheduler-side construct, no request/response
field or query parameter is added.

Security impact
---------------

The operator must secure the tooz backend (ZooKeeper/etcd/Redis).
Its credentials live in the scheduler config file. No new OpenStack
credentials are introduced.

Notifications impact
--------------------

None.

Other end user impact
---------------------

None.

Performance Impact
------------------

Each scheduler keeps a connection and heartbeat to the tooz backend.
Ring computation is in-memory and cheap. Per-job lock acquisition adds
negligible latency relative to a backup. All members share a
client_id and poll for the same jobs, so API polling load scales
with cluster size.

Each scheduler frequently polls api, that might cause a performance
impact.

Other deployer impact
---------------------

Adds a ``[coordination]`` config section to freezer-scheduler:

.. code-block:: ini

   [coordination]
   backend_url = <None>          # e.g. redis://host:6379
   heartbeat_interval = 5

The cluster identity is the existing client_id (set in the
scheduler config); all members of a cluster are configured with the
same value. Clustered mode is enabled by setting ``backend_url``. If
``backend_url`` is unset the scheduler runs in standalone mode as
today. Operators must deploy a tooz backend when clustered mode is
used.

Developer impact
----------------

None.

Upgrade impact
--------------

None. There is no schema migration. Existing schedulers without a
coordination backend configured are unaffected and continue to run
standalone.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
    afiliipek (Alicja Filipek)

Feature Liaison
---------------

Feature liaison:
    noonedeadpunk (Dmitriy Rabotyagov)

Work Items
----------

* freezer-scheduler: add ``coordination`` module wrapping tooz, with
  ``[coordination]`` config options.
* freezer-scheduler: tooz group membership keyed on client_id with
  a unique per-process member id (``<hostname>-<pid/random>``)
  (join/leave/heartbeat with membership callbacks).
* freezer-scheduler: hash-ring ownership and per-job lock around
  execution.
* Docs: operator guide for clustered mode (running N schedulers with a
  shared client_id plus a tooz backend).

Dependencies
============

* `tooz`_ — added to freezer-scheduler requirements.
* A tooz-supported backend (ZooKeeper/etcd/Redis) when clustered mode
  is enabled.

Testing
=======

* Unit tests with a mocked tooz driver: ring agreement across members,
  rebalance on join/leave, only the owner runs a job.
* Functional tests with the tooz ``zake`` driver for membership and
  locking without external services.
* Manual test with Redis: kill the owner mid-run and confirm a peer
  takes over with no duplicate concurrent execution.
* Member id uniqueness: two schedulers with a colliding member id are
  handled safely (duplicate join rejected, no ring collapse).
* Backend outage: with the tooz backend unreachable, members skip job
  execution rather than running everything.

Documentation Impact
====================

Coordination section covering what the clustered mode is, how it works,
what problems it solves, when to use it and how to configure it.

References
==========

* `tooz documentation <https://docs.openstack.org/tooz/latest/>`_
* `cinder active-active - job distribution
  <https://specs.openstack.org/openstack/cinder-specs/specs/ocata/ha-aa-job-distribution.html>`_
* `cinder - tooz locks
  <https://specs.openstack.org/openstack/cinder-specs/specs/mitaka/ha-aa-tooz-locks.html>`_
