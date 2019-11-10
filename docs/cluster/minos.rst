minos
#####

We use afq984/minos (`github <https://github.com/afq984/minos>`_) as the user authentication service across cluster nodes.

.. contents:: :depth: 2

Prerequisites
=============

Install packages required for building minos.
::

	yum install rpm-build gcc meson glib2-devel zeromq-devel

Build
=====

.. hint::

	We will place ``minos-0.0.16-1.el7.x86_64.rpm`` in the disk we bring to contest.
	This file should also appears in ``qct1``'s boot disk.

Download minos.spec (`link <https://github.com/afq984/minos.spec/blob/master/minos.spec>`_) and run
::

	rpmbuild --undefine=_disable_source_fetch -ba minos.spec

Where the ``--undefine=_disable_source_fetch`` is to allow rpmbuild automatically download minos source code.


After build complete successfully, install the generated .rpm file in ``$HOME/rpmbuild/RPMS/x86_64``.
::

	rpm -ivh $HOME/rpmbuild/RPMS/x86_64/minos-0.0.16-1.el7.x86_64.rpm

Configuration
=============

Master
^^^^^^

Change the ``Address`` in ``/etc/minos.d/server.conf`` to master node's address in the local network.
::

	Address=tcp://10.19.0.1:6148

.. hint::
	There are many other options in ``server.conf``, which allow you to specify which users and groups you would like to make available to the client machines.

Start and enable ``minos-server.service``.
::

	systemctl enable minos-server.service --now

Worker
^^^^^^

Change the ``Address`` in ``/etc/minos.d/client.conf`` to master node's address in the local network.
::

	Address=tcp://10.19.0.1:6148

Start and enable ``minos-client.service``.
::

	systemctl enable minos-client.service --now

Edit ``/etc/nsswitch.conf`` and add ``minos`` to ``passwd``, ``group`` and ``shadow``.
::

	# /etc/nsswitch.conf
	passwd:     files sss minos
	shadow:     files sss minos
	group:      files sss minos

.. warning::

	Password change only takes effect on master node. Changing password on worker nodes may lead to unexpected behavior.
