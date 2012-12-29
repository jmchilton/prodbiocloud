.. Production Galaxy Instances with ClouMman and CloudBioLinux documentation master file, created by
   sphinx-quickstart on Wed Nov 28 21:06:46 2012.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

.. TODO: Normalize user-data references

========================================================================
Production Galaxy Instances with CloudMan and CloudBioLinux
========================================================================

.. _EC2: http://aws.amazon.com/ec2/
.. _Galaxy: http://galaxyproject.org/
.. _CloudMan: https://bitbucket.org/galaxy/cloudman/
.. _CloudBioLinux: https://github.com/chapmanb/cloudbiolinux/
.. _nginx: http://nginx.org/
.. _OpenStack: http://www.openstack.org/

Introduction
------------

Typically, `CloudMan`_ is used to spin up a dedicated `Galaxy`_ virtual
cluster for a single user or small group on Amazon `EC2`_. This document
outlines methods for deploying customized `Galaxy`_ instances that are more
suitable for production environments with large user bases - these
enhancements include enabling load balancing, external authentication, SSL,
reporting, and taking advantage of external resources such as dedicated file
and database servers and compute resources outside of the cloud.

Some of these enhancements can be used on `EC2`_ with updated stock
`CloudMan`_ images, however some require rebuilding a `CloudMan`_ image using
`CloudBioLinux`_ (as is required when utilizing private cloud resources).
:ref:`getting_started` therefore provides some guidance on building custom
`CloudMan`_ images with `CloudBioLinux`_. Readers without `CloudBioLinux`_
familiarity should review this appendix should be reviewed before continuing.

.. include:: production.rst

.. _getting_started:

Appendix I: Getting Started with `CloudBioLinux`_
-------------------------------------------------

.. include:: getting_started.rst

.. _configuring_nginx_conf:

Appendix II: Configuring nginx.conf
----------------------------------------------

.. include:: nginx.rst

.. _configuration_directory:

Appendix III: Galaxy Configuration Directories
----------------------------------------------

.. include:: conf_dirs.rst
