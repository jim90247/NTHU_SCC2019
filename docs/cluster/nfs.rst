NFS
###

We use Network File System (NFS) to share the files on master node to worker nodes.

.. contents:: :depth: 2

Before Start
============

Install packages.
::

	yum install nfs-utils

Server
======

Configure the directory to shared on NFS. Edit ``/etc/exports``.
::

	/home 10.19.0.0/24(rw,async,no_root_squash) 10.19.1.0/24(rw,async,no_root_squash)
	/opt 10.19.0.0/24(rw,async,no_root_squash) 10.19.1.0/24(rw,async,no_root_squash)

``no_root_squash``  make root on NFS clients have same access rights as NFS server's root.

Enable and start system service.
::

	systemctl enable nfs-server --now

Client
======

Mount NFS file systems. Edit ``/etc/fstab``. (Mount via infiniband).
::

	10.19.1.1:/home /home nfs4 nofail,soft,intr,bg 0 0
	10.19.1.1:/opt /opt nfs4 nofail,soft,intr,bg 0 0

Activate the mount settings.
::

	mount -a

.. note::
	``tmpfs`` in CentOS are ramdisks. If exporting ramdisk over NFS, need another export flag: ``fsid=xxx``, where ``xxx`` can be any integer greater than 0.
	::

		/ramdisk 10.19.1.0/24(rw,async,no_root_squash,fsid=1)
