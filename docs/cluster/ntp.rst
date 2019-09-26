NTP
###

NTP is used to synchronize time between cluster nodes. We configure master node to be the NTP server.

.. contents:: :depth: 2

Before start
============

Install NTP package.
::

	yum install ntp
	
Master
======

If public NTP servers are accessible, we can use them as synchronization source.

Edit ``/etc/ntp.conf`` to contain the following content only (you should backup the old one).
::

	server time.stdtime.gov.tw
	server clock.stdtime.gov.tw

Multiple NTP servers are encouraged.

If master node cannot access internet, we have to set up our own local NTP server. Add the following content into ``/etc/ntp.conf``.
::

	server 127.127.1.0
	fudge 127.127.1.0 stratum 10
	
Finally, enable and start service.
::

	systemctl enable ntpd --now
	
Worker
======

We use **chrony** as an replacement of *ntpdate*.

Install chrony.
::

    yum install chrony -y

Edit config file ``/etc/chrony.conf``. Adjust NTP servers we would like to use. Here we use the master node of our cluster as NTP server.
::
    ...
    server 10.18.0.1
    ...

Start system service.
::

    systemctl start chronyd --now

Check synchronization status with NTP server.
::

    $ chronyc tracking
    Reference ID    : 0A130001 (10.18.0.1)
    Stratum         : 12
    Ref time (UTC)  : Thu Sep 26 18:48:33 2019
    System time     : 0.000000531 seconds fast of NTP time
    Last offset     : +0.000004994 seconds
    RMS offset      : 0.000088117 seconds
    Frequency       : 0.086 ppm fast
    Residual freq   : +0.001 ppm
    Skew            : 0.076 ppm
    Root delay      : 0.000194579 seconds
    Root dispersion : 0.010986784 seconds
    Update interval : 64.9 seconds
    Leap status     : Normal