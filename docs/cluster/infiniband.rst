InfiniBand
##########

.. contents:: :depth: 2

InfiniBand is a kind of interconnection widely used in supercomputers. It features *high throughput* and **low latency**. We use InfiniBand cards manufactured by `Mellanox`_.

.. _Mellanox: https://www.mellanox.com/

Driver and system setup
=======================

Download latest MLNX_OFED driver (`driver download link`_), decompress it and ``cd`` into the directory.

.. _driver download link: http://www.mellanox.com/page/mlnx_ofed_matrix?mtag=linux_sw_drivers


.. note::
	There are open-source drivers available. We use the driver provided by Mellanox here.

Before installation proceeds, install some required libraries first.
::

	yum install tcl tk

Run install script. This might take some time.
::

	./mlnxofedinstall

If required, unload old driver and load new driver.
::

	modprobe -rv ib_isert rpcrdma ib_srpt
	/etc/init.d/openibd restart

Start opensm (*InfiniBand subnet manager*).
::

	systemctl enable opensmd --now

.. warning::
	Start ``opensmd`` on one of the cluster node only. If multiple nodes start ``opensmd`` at the same time, it may cause problem.

Check InfiniBand status.
::

	ibstat
	# State: Active
	# Physical state: LinkUp
	
	ibstatus

IPoIB
=====

We can use InfiniBand as usual network interface.

ifconfig
^^^^^^^^

Create a ifconfig script in ``/etc/sysconfig/network/scripts``.
::

	TYPE=InfiniBand
	BOOTPROTO=static
	IPV6_AUTOCONF=no
	NAME=ib0
	DEVICE=ib0
	ONBOOT=yes
	IPADDR=10.18.18.1
	PREFIX=24
	CONNECTED_MODE=yes
	MTU=65520
	
Here we enable **connected mode** to maximize performance.

Restart network.
::

	systemctl restart network

systemd-networkd
^^^^^^^^^^^^^^^^

Create a config file in ``/etc/systemd/network``.
::

	[Match]
	Name=ib0

	[Network]
	Address=10.18.18.1/24

Restart ``systemd-networkd``.
::

	systemctl restart systemd-networkd

To enable connected mode, we can use this `AUR package`_. Remove the ``rdma.service`` dependency in ``ipoibmodemtu.service``, since we are using Mellanox's driver.

.. _AUR package: https://aur.archlinux.org/packages/ipoibmodemtu/

Copy the files to their specified path, then start systemd service.
::

	cp ipoibmodemtu /usr/bin/ipoibmodemtu
	cp ipoibmodemtu.conf /etc/ipoibmodemtu.conf
	cp ipoibmodemtu.service /usr/lib/systemd/system/ipoibmodemtu.service
	systemctl daemon-reload
	systemctl enable ipoibmodemtu.service --now

Mellanox OFED GPUDirect RDMA
============================

Not included in Mellanox driver. Get the source from github.
::

	git clone https://github.com/Mellanox/nv_peer_memory.git
	git checkout d9a0a126bc31279e63e4b47bfb59ee767cea7aad

.. caution::
    As of 11/11/2019, we found the latest version of nv_peer_mem will fail on our cluster (*art*). That's why we use an older commit.


Build kernel module.
::

	./build_module.sh

Install using ``rpm``.
::

	rpmbuild --rebuild /tmp/nvidia_peer_memory-*.src.rpm
	rpm -ivh $HOME/rpmbuild/RPMS/x86_64/nvidia_peer_memory-*.x86_64.rpm

Start system service.
::

	systemctl enable nv_peer_mem --now
