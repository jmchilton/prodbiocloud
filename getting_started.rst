
There are a few key configuration files used to configure
`CloudBioLinux`_ + `CloudMan`_. These include:

.. _fabricrc:

``fabricrc.txt``
  `Fabric`_ properties used to configure `CloudBioLinux`_. For examples see the
  `CloudBioLinux default <https://github.com/chapmanb/cloudbiolinux/blob/master/config/fabricrc.txt>`_ or the version used for my `CloudMan OpenStack bootstrap scripts <https://github.com/jmchilton/cloudman_openstack_bootstrap/blob/master/fabricrc.txt.sample>`_

``userData.yaml``
  A YAML configuration file used by `CloudMan`_ to configure your instance(s) at
  startup.

``nginx.conf``
  Base configuration file for nginx_ server. See
  :ref:`configuring_nginx_conf` for description of how to replace the default
  contents of this file either at image build time or at instance boot time.

Checking Out CloudBioLinux
~~~~~~~~~~~~~~~~~~~~~~~~~~

``git`` should be used obtain ``CloudBioLinux``.

::

    git clone git://github.com/chapmanb/cloudbiolinux.git

If additional `CloudBioLinux`_ development is planned (installation of custom
packages, services, etc...), it may be preferable to `fork CloudBioLinux`__ first and then
checkout `CloudBioLinux`_ from the forked repository.

__ fork_Cloudbiolinux_

.. _fork_CloudBioLinux: https://github.com/chapmanb/cloudbiolinux/fork


Installing CloudBioLinux
~~~~~~~~~~~~~~~~~~~~~~~~

Prerequistes for installing `CloudBioLinux`_ include the following:

- A customized `fabricrc`_ file as described above (referred to as ``/path/to/fabricrc.txt``). 
- A running Ubuntu instance on your cloud platform. This document assumes key-pairs and security groups so that the instance (``test1.example.org``) is accessible via the private key ``/path/to/private_key``.
- Required filesytem mounts (likely at least `/mnt/galaxyData` and `/mnt/galaxyTools`).

The specifics of these steps are beyond the scope of this document and will
vary from setup to setup.

.. TODO: Find a good link for this

CloudMan Flavor
+++++++++++++++

Once you have booted up an Ubuntu instance in your cloud environment,
``CloudBioLinux`` can be installed using `fabric`_ via the following command::

    fab -u ubuntu -c /path/to/fabricrc.txt -i /path/to/private_key -H test1.example.org install_biolinux:flavor=cloudman

This version of the install command is the simplest, and it will install some
base packages and configure `CloudMan`_. This is good for initial testing.

Ultimately, though you will likely want to install needed bioinformatics
applications also - the following two subsections describe approaches to
accomplish this.

Flavorless
++++++++++

::

    fab -u ubuntu -c /path/to/fabricrc.txt -i /path/to/private_key -H test1.example.org install_biolinux

This second version of the ``install_biolinux`` command (no flavor specified)
will install all of `CloudBioLinux`_, which will include many packages you
may not need for Galaxy, such as R libraries and desktop applications.

These packages will be installed into the instance's ``/usr`` directory,
either using fabric commands defined in ``cloudbiolinux/cloudbio/custom`` or
native pacakges.

If installing all of `CloudBioLinux`_ is not desired, one can comment out
lines in ``cloudbiolinux/config/main.yaml`` and
``cloudbiolinux/config/custom.yaml`` and other files in that directory to
prune the list of applications and libraries that get installed.

CloudMan + Tools Flavor
+++++++++++++++++++++++

As mentioned above, the flavorless install will install all packages into
``/usr``. This allows applications other than Galaxy to utilize these programs,
but this has down sides. 

With every program on Galaxy's path by default, it may be difficult to isolate
certain types of problems. More importantly however, Galaxy cannot then easily
target multiple versions of the same program with different tool wrappers.

``CloudBioLinux`` can install specific Galaxy tools into a Galaxy `tool
dependency directories
<http://wiki.galaxyproject.org/Admin/Config/Tool%20Dependencies>`_ along with
required ``default`` symbolic links and ``env.sh`` files. To install
CloudBioLinux this way, the following command can be used:

::

    fab -u ubuntu -c /path/to/fabricrc.txt -i /path/to/private_key -H test1.example.org install_biolinux:flavor=cloudman_and_galaxy

This approach also requires updating the target `fabricrc`_ file to instruct
`CloudBioLinux`_ to install Galaxy dependencies.

::

    galaxy_install_dependencies = true

The list of tools and versions that is installed can be found in
``cloudbiolinux/contrib/flavor/cloudman/tools.yaml <https://github.com/chapman
b/cloudbiolinux/blob/master/contrib/flavor/cloudman/tools.yaml>``. One can
modify that file directly or specify an entirely new file by setting the
``galaxy_tools_conf`` property in `fabricrc`_.

*Warning*: Installing Galaxy tools in this manor is not as well supported as the
full `CloudBioLinux`_ approach. This approach is my (John Chilton) pet
project, and my day job is proteomics not genomics so many of the tools are
suitable for `Galaxy`_ circa mid-2012, not an updated `Galaxy`_ with Cufflinks 2,
Tophat 2, etc.... I am happy to accept pull requests to integrate newer
versions of these tools however.


Save the Image
~~~~~~~~~~~~~~

The `CloudBioLinux`_ install will setup a script that get executed at instance
startup to launch `CloudMan`_. So the next step is to save this image and boot
it up with the required `CloudMan`_ userdata. Details of how to do this will
vary between Cloud platforms.

.. _fabric: http://docs.fabfile.org/
