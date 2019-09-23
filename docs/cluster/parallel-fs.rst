Parallel File System
####################

In SCC19, a storage system benchmark is introduced: *IO-500*.

While NFS provides a shared file system across cluster nodes, its main focus is not to improve the storage system's throughput. This is where parallel file systems like Lustre, GPFS and BeeGFS come into place. Parallel file systems store data across multiple nodes, and provide simultaneous I/O operations on multiple storage nodes.

.. contents:: :depth: 3

BeeGFS
******

More information on installation and setup can be found at `BeeGFS Wiki <https://www.beegfs.io/wiki/InstallationSetupGuide>`_.

Installation
============

Installation steps can be found at `this wiki page <https://www.beegfs.io/wiki/ManualInstallWalkThrough>`_.

.. note::
    When specifying paths to BeeGFS service during configuration, use **absolute path** instead of *relative path*.

.. hint::
    Existing file systems (such as ext4, xfs) can be used as data storage for BeeGFS services.

Repository
----------

* ``beegfs-mgmtd``: Management Server (one node)

	* Manages configuration and group membership
	* Hostname or IP address must be known by other nodes at service start time

* ``beegfs-meta``: Metadata Server (at least one node)

	* Stores directory information and allocates file space on storage servers

* ``beegfs-storage``: Storage Server (at least one node)

	* Stores raw file contents

* ``beegfs-client`` and ``beegfs-helperd``: Client

	* Kernel module to mount the file system
	* Requires userspace helper daemon for logging and hostname resolution

* ``libbeegfs-ib``: RDMA support

	* libraries for RDMA support for Metadata and Storage Services

* ``beegfs-utils``: BeeGFS utilities for administrators

	* ``beegfs-ctl`` tool for command-line administration
	* ``beegfs-fsck`` tool for file system checking
	* Several small helper scripts

Some optional packages are omitted here.

Client Kernel Module
--------------------

To enable InfiniBand & RDMA support, change ``buildArgs`` in ``/etc/beegfs/beegfs-client-autobuild.conf``:
::

    buildArgs=-j8 BEEGFS_OPENTK_IBVERBS=1 OFED_INCLUDE_PATH=/usr/src/ofa_kernel/default/include

Then run 
::

    /etc/init.d/beegfs-client rebuild
