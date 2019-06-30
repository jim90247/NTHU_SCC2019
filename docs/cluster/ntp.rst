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

Workers can use ``ntpdate`` to synchronize time with master.
::

	ntpdate <server IP>
	
If we want to sync with master periodically, we can put it into ``crontab``.