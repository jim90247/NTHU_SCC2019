NIS
###

We use Network Information Service (NIS) to let user on master node access worker nodes.

.. contents:: :depth: 2

Before start
============

Install NIS packages.
::

	yum install yp-tools ypbind ypserv
	
Server
======

Setup
^^^^^

Append following contents to ``/etc/sysconfig/network``.
::

	NISDOMAIN=<domainname>
	YPSERV_ARGS="-p 1011"

Edit ``/etc/ypserv.conf`` to set NIS access permissions. For example, only *localhost* and *10.18.0.0/24* are allowed in the following settings.
::

	127.0.0.0/255.0.0.0 : * : * : none
	10.18.0.0/255.255.255.0 : * : * : none
	* : * : * : deny

Set hostname-IP mapping in ``/etc/hosts``. Make sure all the hostnames (server/client) exist in ``/etc/hosts`` and IP settings are correct.
::

	# check the output
	cat /etc/hosts
	hostname
	
Start services.
::

	systemctl enable ypserv --now
	systemctl enable yppasswdd --now
	
Initialize NIS database. Press *Ctrl-D* when "next host to add" appears.
::

	/usr/lib64/yp/ypinit -m

Update database
^^^^^^^^^^^^^^^

User can change their account settings at **master node**.

In the future, if anything related to user account changes (e.g. change password, shell, group info updated), we have to update server's database.
::

	make -C /var/yp

.. note::
	``yppasswd`` seems not working at the time this document is written.

Client
======

Setup
^^^^^

Use a GUI tool ``setup`` to configure.
::

	setup

In the GUI, mark *Use NIS*.

Fill in the fields with corresponding fields.

* Domain: <domainname> (the <domainname> set at server)
* Server: 10.18.0.1 (server IP)

If configured successfully, the GUI will quickly return to initial window.

Test
^^^^

::

	yptest

Warning at test 3 and 4 can be ignored.

::

	systemctl status ypbind
	
If ``ExecStartPre=/usr/sbin/setsebool allow_ypbind=1 (code=exited, status=1/FAILURE)`` appears, just ignore it (since selinux is disabled).

Finally, make sure user can ``ssh`` to NIS clients. If not, check the above settings and selinux status (must be *disabled*).