Yum package manager
###################

.. contents:: :depth: 2

Basic usage
===========

Install packages.
::

	yum install [package_name]

Update packages. If no ``package_name`` is specified, all packages will be updated.
::

	yum update [package_name]

Some useful repository
======================

epel
^^^^

Extra Packages for Enterprise Linux (EPEL).
::

	yum install epel-release

ius
^^^

This repository contains packages newer than CentOS-Base repository. (e.g. tmux2, git2)

::

	yum install https://centos7.iuscommunity.org/ius-release.rpm

.. note::
	You can also copy ``art:/etc/yum.repos.d`` (the whole directory) to our cluster and remove unused repositories.

Local repository
================

To fix packages' and their dependencies' version, we tend to keep copies of repositories. Also, there might not have internet access during competition, local repositories can make package manager like yum still works.

Install
^^^^^^^

First install some useful tools.
::

	yum install yum-utils createrepo

Synchronize packages on mirror sites to local.
::

	reposync --gpgcheck -l --repoid=<repo> --downloadcomps --download-metadata -a x86_64 --newest-only

Set up repository information.
::

	createrepo -v <path_to_repository> [-g comps.xml]
	# if comps.xml exists in repository path, add the -g option

Use
^^^

Start HTTP server in the repository directory.
::

	$ ls /opt/repo
	base extras updates
	$ darkhttpd /opt/repo

Repository configurations locate at ``/etc/yum.repos.d``. Changes the ``baseurl`` to local http server. Below is an example.
::

	[base]
	name=CentOS-$releasever - Base
	baseurl=http://localhost/base/
	gpgcheck=0

.. note::
	If we want to disable *fastest mirror* plugin (all repositories are local), we can edit ``/etc/yum/pluginconf.d/fastestmirror.conf``.

	::

		enabled=0
