
The core of this document relates to configuring various aspects of
CloudMan/Galaxy geared toward large, long running production instances but
first one needs to configure a CloudMan machine image. This section outlines
how to do that both in an abstract way applicable for any cloud environment
and then specially for OpenStack using the set of scripts used maintain to
manage the CloudMan environment at MSI.

There are a few key configuration files used to configure
CloudBioLinux+CloudMan. These are

.. _fabricrc:

``fabricrc.txt``

  Fabric properties used to configure `CloudBioLinux`. For examples see the
  `CloudBioLinux default <https://github.com/chapmanb/cloudbiolinux/blob/maste
  r/config/fabricrc.txt>`_ or the version used for `my CloudMan OpenStack
  bootstrap scripts <https://github.com/jmchilton/cloudman_openstack_bootstrap
  /blob/master/fabricrc.txt.sample>`_

``userData.yaml``

  YAML configuration file used by `CloudMan` to configure your instance(s) at
  startup.

``nginx.conf``

  Base configuration file for nginx_ server. See
  :ref:`configuring_nginx_conf` for description of how to replace the default
  contents of this file either at image build time or at instance boot time.

.. _nginx: http://nginx.org/

Checking Out CloudBioLinux
~~~~~~~~~~~~~~~~~~~~~~~~~~

One can use ``git`` to checkout ``CloudBioLinux``.

::

    git://github.com/chapmanb/cloudbiolinux.git

If you forsee considerable CloudBioLinux development (installation of custom
packages, services, etc...), you will likly want to `fork CloudBioLinux`__ first and then
checkout your own copy.

__ fork_Cloudbiolinux_

.. _fork_CloudBioLinux: https://github.com/chapmanb/cloudbiolinux/fork


Installing CloudBioLinux
~~~~~~~~~~~~~~~~~~~~~~~~

In order to install CloudBioLinux, you will need to have the following done:

- Customize a `fabricrc`_ file as described above (referred to as
``/path/to/fabricrc.txt``). 
- Launch a stock Ubuntu instance on your cloud platform with key-pairs and
security groups so that the instance (``test1.example.org``) is accessible 
via the private key ``/path/to/private_key``.
- Mount needed file systems (likely at least `/mnt/galaxyData` and 
`/mnt/galaxyTools`). The specifics of this is beyond the scope of this 
document.

CloudMan Flavor
+++++++++++++++

Once you have booted up an Ubuntu instance in your cloud environment,
``CloudBioLinux`` can be installed using `fabric`_ via the following command::

    fab -u ubuntu -c /path/to/fabricrc.txt -i /path/to/private_key -H test1.example.org install_biolinux:flavor=cloudman

This version of the install command is the simplest, and it will install some
base packages and configure CloudMan. This is good for initial testing.
Ultimately, though you will likely want to install needed bioinformatics
applications also - the following two subsections describe two approaches to
this.

Flavorless
++++++++++

::

    fab -u ubuntu -c /path/to/fabricrc.txt -i /path/to/private_key -H test1.example.org install_biolinux

The second version of the install_biolinux command (no flavor specified) will
install all of ``CloudBioLinux``, which will include many packages you may not
need for Galaxy, such as R libraries and desktop packages.

These packages will be installed into the image's /usr partition, either using
fabric commands defined in ``cloudbiolinux/cloudbio/custom`` or using native pacakges.

If all of ``CloudBioLinux`` is not desired, one can comment out lines in
``cloudbiolinux/config/main.yaml`` and ``cloudbiolinux/config/custom.yaml``
and other files in that directory to prune the list of things that get
installed.

CloudMan + Tools Flavor
+++++++++++++++++++++++

As mentioned above, the flavorless install will install all packages into
/usr. This allows applications other than Galaxy to utilize these programs,
but this has down sides. 

Galaxy's path grows quite complex, always having all programs on it. Galaxy
cannot easily target multiple versions of the same program with different tool
wrappers. 

``CloudBioLinux`` can install specific Galaxy tools into a Galaxy `tool
dependency directories
<http://wiki.galaxyproject.org/Admin/Config/Tool%20Dependencies>`_ along with
required ``default`` symbolic links and ``env.sh`` files. To install
CloudBioLinux this way, the following command can be used:

Update the target `fabricrc`_ file to install Galaxy dependencies.

::

    galaxy_install_dependencies = true


Install `CloudBioLinux`_ using the ``cloudman_and_galaxy`` flavor.

::

    fab -u ubuntu -c /path/to/fabricrc.txt -i /path/to/private_key -H test1.example.org install_biolinux:flavor=cloudman_and_galaxy

The list of tools and versions that is installed can be found in
``cloudbiolinux/contrib/flavor/cloudman/tools.yaml <https://github.com/chapman
b/cloudbiolinux/blob/master/contrib/flavor/cloudman/tools.yaml>``. One can
modify that file directly or specify an entirely new file by setting the
``galaxy_tools_conf`` property in `fabricrc`_.

*Warning*: Installing Galaxy tools in this manor is not as well supported as the
full `CloudBioLinux`_ approach. This approach is my (John Chilton) pet
project, and my day job is proteomics not genomics so many of the tools are
suitable for Galaxy circa mid-2012, not an updated Galaxy with Cufflinks 2,
Tophat 2, etc.... I am happy to accept pull requests to integrate newer
versions of these tools however.


Save the Image
~~~~~~~~~~~~~~

The ``CloudBioLinux`` install will setup a script that get executed at
startup, so to launch CloudMan save this image and boot it up with the
required CloudMan userdata. Details of how to do this will vary between Cloud
platforms.


.. _fabric: http://docs.fabfile.org/
