Miscellaneous
#############

.. contents:: :depth: 2

SELinux
=======

Temporarily disable SELinux:
::

	setenforce 0

To permanently disable SELinux, edit ``/etc/sysconfig/selinux`` and ``/etc/selinux/config``.
::

	SELINUX=disabled

SSH
===

Permit root login from local subnet
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Edit ``/etc/ssh/sshd_config``. (*Match* section overrides default settings.)
::

	PermitRootLogin no
	Match Address 10.18.0.0/24
		PermitRootLogin yes

Login without password
^^^^^^^^^^^^^^^^^^^^^^

To access worker nodes (or from other computers to server) without password, we could to use ssh key as authentication method.

Generate ssh key.
::

	ssh-keygen
	# Press Enter to the end.
	# ssh-keygen -t <key_type> if you want to use methods other than RSA.

For nodes we want to login without password, copy our public key to those nodes.
::

	ssh-copy-id <node>

After copy, try ssh to that node. See if password is still required.

.. note::
	This is required for ``mpirun`` to run across multiple nodes.

Unlimited stack size
====================

Some application might receive ``SIGSEGV`` during execution. This is usually caused by limited stack size.

Getting current limit settings:
::

	ulimit -Ss # soft limit
	ulimit -Hs # hard limit

* soft limit: user can change it by himself/herself.
* hard limit: only root can change.

The stricter restriction applies.

To remove the stack size limit *permanently*, system admin can create a file under ``/etc/security/limits.d``. Here, we name it as ``40-stacksize.conf``.

Fill the file with following content.
::

	# <User> <soft|hard> <target> <limit>
	* soft stack unlimited
	* hard stack unlimited

Software RAID
=============

We use ``mdadm`` to create a *software RAID* system (does not require RAID hardware).

Install from yum.
::

	yum install mdadm

Sample usage
^^^^^^^^^^^^

Recover old RAID.
::

	mdadm --assemble /dev/mdxxx /dev/sda /dev/sdb

Create RAID.
::

	# mdadm --create /dev/mdxxx --level=[RAID level] --raid-devices=[# of disks] /dev/sda /dev/sdb ...
	mdadm --create /dev/md127 --level=0 --raid-devices=2 /dev/sda /dev/sdb

Ansible
=======

Ansible is a tool to efficiently manage multiple cluster nodes at the same time.

Install from yum.
::

	yum install ansible

Configuration file: ``/etc/ansible/hosts``.
::

	[worker]
	node1 ansible_ssh_host=10.18.0.1
	node2 ansible_ssh_host=10.18.0.2

Clustershell
============

Clustershell is another tool to manage multiple cluster nodes.

Install from yum.
::

	yum install clustershell

Cluster node group configuration file: ``/etc/clustershell/groups.d/local.cfg``.
::

	all: art[1-4]
	master: art1
	worker: art[2-4]

To execute a command for certain group, run
::

	# clush -w @<group> [-b [--diff]] [-L] <command>
	clush -w @worker -b --diff "rpm -qa | sort"


* ``-b`` buffers output from each node, output all results after all nodes finish execution. 
* ``--diff`` can compare the results of each node.
* ``-L`` will show outputs in the order of machines' name.
