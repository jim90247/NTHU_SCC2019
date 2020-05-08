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
	NAME=enp1s0f1
	IPADDR0=<public IP>
	IPADDR1="10.18.0.1"
	PREFIX0=24
	PREFIX1=24
	GATEWAY=<gateway>
	DNS1=8.8.8.8
	ONBOOT=yes

For worker nodes, you might not need public IP setting.

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
	Address=<public IP>/24
	Address=10.18.0.1/24
	Gateway=<gateway>
	DNS=8.8.8.8
	
For worker nodes, you might not need public IP setting. 

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

firewalld
^^^^^^^^^
	
Start ``firewalld`` service.
::

	systemctl enable firewalld --now
	
Add firewall rules.
::

	firewall-cmd --set-default-zone=external
	firewall-cmd --permanent --zone=external --change-interface=enp1s0f1
	firewall-cmd --permanent --zone=external --change-interface=enp1s0f1:0
	firewall-cmd --permanent --zone=trusted --add-source=10.18.0.0/24
	firewall-cmd --permanent --zone=trusted --add-source=10.18.18.0/24 # IPoIB
	
nftables
^^^^^^^^

This is an alternative method of using ``firewalld``.

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
	nft add rule nat postrouting ip saddr 10.18.0.0/24 oif enp1s0f1 snat <public IP>

Make settings become permanent.
::

	systemctl enable nftables --now
	
	# After configuring above rules
	nft list ruleset >> /etc/sysconfig/nftables.conf

sysctl
^^^^^^

Add ``net.ipv4.ip_forward=1`` into ``/etc/sysctl.conf``. Then do
::

	sysctl -p
	
to reload sysctl settings.

.. note::
	If NAT is not working and ``libvirtd`` is enabled, try disabling ``libvirtd`` (it uses iptables).

Firewall
========

Control firewalld settings to allow/block connections.

Enable and start ``firewalld`` service.
::

  systemctl enable firewalld --now

Query current configurations.
::

  firewall-cmd --list-all

Query active zones of each network interface.
::

  firewall-cmd --get-active-zones

Query predefined services.
::

  firewall-cmd --get-services

Add a predefined service into some zone.
::

  firewall-cmd --zone=<zone> [--permanent] --add-service=<service>

Query permanent services of some zone.
::

  firewall-cmd --zone=<zone> --permanent --list-serivces

Open custom ports (ports not included in predefined services).
::

  firewall-cmd --zone=<zone> [--permanent] --add-port=X[-Y]/<protocol>
  # example: firewall-cmd --zone=public --add-port=8080/tcp
  # example 2: firewall-cmd --zone=public --add-port=4990-4999/udp

Remove opened services/port.
::

  firewall-cmd --zone=<zone> [--permanent] --remove-service=<service>
  firewall-cmd --zone=<zone> [--permanent] --remove-port=<port>/<protocol>

Allow specific IPs to access certain services.
::

  firewall-cmd --zone=<zone> [--permanent] \
    --add-rich-rule 'rule family="ipv4" source address="<IP>/<mask>" service name="<service>" accept'

Add specific IPs to white list (allow connection to all ports).
::

  firewall-cmd --zone=<zone> [--permanent] \
    --add-rich-rule='rule family="ipv4" source address="<IP>/<mask>" accept'
