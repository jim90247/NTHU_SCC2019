Network
#######

There are many ways to set up network in CentOS. In this section, we demonstrate how to use ``ifconfig`` and ``systemd-networkd`` to setup network in CentOS cluster. Both of them can do same thing, but configuration file for ``systemd-networkd`` is simpler.

.. contents:: :depth: 2

Before start
============

First we disable ``NetworkManager`` and ``firewalld``. This can reduce unnecessary problems.

::

	systemctl disable NetworkManager --now
	systemctl disable firewalld --now

ifconfig
========

Configuration
^^^^^^^^^^^^^

Edit network scripts in ``/etc/sysconfig/network-scripts/ifcfg-<device>``, where ``<device>`` can be something like ``enp1s0f1`` or ``eth0``.

The following changes are required. For the remaining settings, it's up to you.
::

	TYPE=Ethernet
	BROWSER_ONLY=no
	BOOTPROTO=static
	DEFROUTE=yes
	NAME=<interface name>
	IPADDR=10.19.0.1
	PREFIX=24
	GATEWAY=10.19.0.1
	DNS1=8.8.8.8
	ONBOOT=yes

For master node, you might not need GATEWAY IP setting.

.. note::
	Make sure all nodes' IP are distinct.

Activate
^^^^^^^^

Enable and start the system service.
::

	systemctl enable network
	systemctl restart network

systemd-networkd
================

Disable ifconfig network service.
::

	systemctl disable network --now


Configuration
^^^^^^^^^^^^^

Edit configuration file ``/etc/systemd/network/<device>.network``, where ``<device>`` can be something like ``enp1s0f1`` or ``eth0``.

::

	[Match]
	Name=<device>

	[Network]
	Address=10.19.0.1/24
	Gateway=10.19.0.1
	DNS=8.8.8.8
	IPForward=ipv4

For master node, you might not need ``Gateway`` IP setting.

For worker node, you don't need ``IPForward`` setting.

.. important::
	The ``<device>`` in file name can be anything, but the ``<device>`` in configuration file must be the same as network interface name. You can find it using command ``ip link``.


Activate
^^^^^^^^

Enable and start ``systemd-resolved``.
::

	systemctl enable systemd-resolved --now
	rm -f /etc/resolv.conf
	ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf

Enable and start ``systemd-networkd``.
::

	systemctl enable systemd-networkd --now


NAT
===

After setting up the network, you might found that worker nodes cannot access internet (only master node can). What you need is to set up NAT service *on master node*.

.. important::
	Perform following operations on **master node** only.

nftables
^^^^^^^^

Install ``nftables``.
::

	yum install nftables

Add SNAT (source NAT) rules.

::

	# create table with chains
	nft add table nat
	nft add chain nat prerouting { type nat hook prerouting priority 0 \; }
	nft add chain nat postrouting { type nat hook postrouting priority 100 \; }

	# add rule
	nft add rule nat postrouting ip saddr 10.19.0.0/24 oif enp1s0f1 snat <public IP>

Make settings become permanent.
::

	systemctl enable nftables --now

	# After configuring above rules
	nft list ruleset >> /etc/sysconfig/nftables.conf

sysctl
^^^^^^

If you already set ``IPForward`` in systemd-networkd configuration file, you may skip this step.

Add ``net.ipv4.ip_forward=1`` into ``/etc/sysctl.conf``. Then do
::

	sysctl -p

to reload sysctl settings.

.. note::
	If NAT is not working and ``libvirtd`` is enabled, try disabling ``libvirtd`` (it uses iptables).
