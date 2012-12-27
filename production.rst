Splitting Galaxy into Multiple Processes
----------------------------------------

`CloudMan`_'s default configuration launches just one `Galaxy`_ process. To
really scale up the number of concurrent users `Galaxy`_ can handle it
must be split into multiple processes as outlined and documented `here
<http://wiki.galaxyproject.org/Admin/Config/Performance/Web%20Application%20Scal
ing>`_::

    galaxy_conf_dir: /opt/galaxy/web/conf.d

    configure_multiple_galaxy_processes: True
    web_thread_count: 2
    handler_thread_count: 2

When these options are enabled, CloudMan will rewrite the body of the
``upstream galaxy_app {...}`` to load balance web traffic accross the
number of web threads you specify. However, when using multiple Galaxy
processes job admin functionality needs to be routed to Galaxy's job
manager, this can be done by updating your nginx.conf file to add the
following ``location /admin/jobs`` subsection shown below::

    location / {
        ...

        location /admin/jobs {
           proxy_pass  http://localhost:8079;
        }
    }

External Authentication (LDAP)
------------------------------

Galaxy requires a proxy web server (in this case nginx) to enable external
authentication. Nginx can be configured to use LDAP authentication by modifing
nginx.conf as follows.

Modify ``http`` section of ``nginx.conf`` with LDAP connection information.

::

    http {
    
        auth_ldap_url ldap://ldap.example.com/dc=example,dc=com?uid?sub?(objectClass=person);
        #auth_ldap_binddn cn=nginx,ou=service,dc=example,dc=com;
        #auth_ldap_binddn_passwd mYsUperPas55W0Rd         
        #auth_ldap_group_attribute uniquemember; # default 'member'
        #auth_ldap_group_attribute_is_dn on; # default on
    
        ...
    
    }

Modify root location of `nginx.conf` to require authentication and pass
`REMOTE_USER` along to Galaxy.

::

    location / {
        auth_ldap_require valid_user;
        auth_ldap "LDAP Auth Source Description";
        proxy_set_header REMOTE_USER $remote_user;


        proxy_pass  http://galaxy_app;
        proxy_set_header   X-Forwarded-Host $host;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_set_header   X-URL-SCHEME https;

      ...
    }
    
    # For API access, set REMOTE_USER if available so Galaxy
    # session based requests are let through, if REMOTE_USER is not
    # available pass the request through and let Galaxy determine
    # if a key is present and valid.
    location  /api {           
        proxy_set_header REMOTE_USER $remote_user;
        proxy_pass  http://galaxy_app;
        proxy_set_header   X-Forwarded-Host $host;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
    }

Additionally, nginx needs to be compiled with LDAP support. This can be done
while building a CloudBioLinux image, to enable this simply add the following
option to your ``fabricrc`` file::

    nginx_enable_module_ldap = true

Finally, Galaxy also must be configured to use the information that will be
passed through by nginx. The method outlined here to do this involves setting
up a Galaxy configuration directory (TODO:link_to_pull_request) via CloudMan
and then set the Galaxy options use_remote_user, remote_user_maildomain,
remote_user_logout_href, and require_login appropriately.::

    galaxy_conf_dir: /opt/galaxy/web/conf.d
    galaxy_universe_use_remote_user: True
    galaxy_universe_remote_user_maildomain: <domain_name (e.g. example.org)>
    galaxy_universe_remote_user_logout_href: https://logout@<galaxy_url>/
    galaxy_universe_require_login: True

If you are using external authentication in this mannor it is also
likely a good idea to enable SSL.


Enable SSL
----------

Your cloud security group will likely block port ``443`` by
default. This must be opened.

If you are using Amazon EC2, when following the instructions on the
`CloudMan wiki site <http://wiki.galaxyproject.org/CloudMan>`_, be sure
to add the HTTPS inbound rule in addition to the HTTP one mentioned.

Instructions for opening this port on private clouds will vary, the
following command for instance will open it for OpenStack.::

    nova secgroup-add-rule <security_group> tcp 443 443 0.0.0.0/0

CloudMan will need to setup the desired SSL key and cert before nginx
starts up. The CloudMan augmentations including::

    conf_files:
       - path: /usr/nginx/conf/key
         content: <base64 encoding of key>
       - path: /usr/nginx/conf/cert
         content: <base64 encoding of cert>


Reports Server
--------------

The Galaxy reports webapp is a small webapp that runs in parallel to
Galaxy and provides a wealth of valuable data on every job that Galaxy
has run as well as disk usage accounting, etc....

CloudMan can now enable the reports application by simply adding it to
the list of services.::

    services:
      - name: Galaxy
      - name: GalaxyReports
      - name: Postgres


External Postgres Server
------------------------

When deploying to Amazon, running a Postgres server right on the
CloudMan/Galaxy head node makes a lot of sense. But for private cloud
deployments, many institutions may already have well optimized, well
maintained production Postgres servers.

To disable the Postgres server, simply manually specify the list of
services CloudMan should start and exclude Postgres. For instance::

    services:
      - name: Galaxy
      - name: GalaxyReports

Then Galaxy must simply be configured to use your external postgres
server, this can be done by passing it in via the userdata variable
``galaxy_universe_database_connection``.::

    galaxy_universe_database_connection: postgres://user:password@host:port/schema


External File Server
--------------------

Two CloudMan user data options - ``master_prestart_commands`` and
``workder_prestart_commands`` - can be specified to run arbitrary shell
commands before CloudMan starts up Galaxy on the master node or runs jobs on
newly booted worker nodes.

The following commands mount Galaxy's data partition from an NFS export on
``spider.msi.umn.edu`` and a read-only partition from an NFS export on
``buzzard.msi.umn.edu`` (we use the second to store bio data such NGS indices,
etc...).::

    master_prestart_commands:
      - "mkdir -p /mnt/galaxyData"
      - "mount -t nfs4 -o sec=sys spider.msi.umn.edu:/export/galaxyp /mnt/galaxyData/"
      - "mkdir -p /project/db"
      - "mount -t nfs4 -o ro buzzard.msi.umn.edu:/zprod2/misc/db /project/db/"
    worker_prestart_commands:
      - "mkdir -p /mnt/galaxyData"
      - "mount -t nfs4 -o sec=sys spider.msi.umn.edu:/export/galaxyp /mnt/galaxyData/"
      - "mkdir -p /project/db"
      - "mount -t nfs4 -o ro buzzard.msi.umn.edu:/zprod2/misc/db /project/db/"



Running Jobs on External Compute Resources
------------------------------------------

The method I will outline here involves the `LWR`_ job runner. 
The LWR job runner is a Galaxy job runner and corresponding server-side
application that can run jobs a server remote to the Galaxy host but without
requiring the same file systems to be mounted on both hosts. It does this by
transferring all input files to the remote host, rewritting paths in the
Galaxy command-line as well as `configfile` s, running the job remotely, and
then transferring the outputs back to the Galaxy host upon completion.

This is being used at MSI to run jobs orginating from an ephermeral Galaxy
host in our OpenStack cloud on a permant Windows host outside the cloud. This
is a useful tool for purchased node-locked and/or Windows only software.

In order to support this use case, CloudMan has been augmented to allow
specifing tool runners via user data. The following piece of userdata is used
to tell CloudMan to configure Galaxy to run ``proteinpilot`` jobs on the
remote Windows host ``cobalt.msi.umn.edu`` using the LWR job runner.::

    galaxy_tool_runner_proteinpilot: "lwr://https://secretkey123@cobalt.msi.umn.edu:8913"

The secret key seen here is used to authorize Galaxy to submit jobs to the
remote LWR host, and https is used to secure transport. Please consult the LWR
documentation and source for details.

Backend implementations for LWR targetting DRMAA and/or PBS are being
developed. Progress can be tracked by following the LWR on 
`Bitbucket <https://bitbucket.org/jmchilton/lwr>`_.

An Aside
~~~~~~~~

It MAY well be possible to configure Galaxy's standard job runner to submit
Galaxy jobs directly from say a cloud host to a traditional, if all of the
file systems are mounted similarly and the remote server has a user that can
run jobs with pid 1001 (the CloudBioLinux generated pid for Galaxy).

If this does work, one could imagine running jobs of type ``tool_x`` via the
PBS host on ``compute.example.com`` by passing along the following user data
to CloudMan at deploy time::

    galaxy_universe_start_job_runners = drmaa, pbs  # Make sure drmaa is still enabled for Cloud-targetted job
    galaxy_tool_runner_tool_x = pbs://compute.example.com/

At this point this is all untested speculation, but hopefully additional
testing will be done and this documentation updated. If you have tried this
and have advice `let me know <mailto:jmchilton@gmail.com>`_

.. _LWR: https://lwr.readthedocs.org/
