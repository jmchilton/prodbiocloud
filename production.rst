----------------------------------------
Splitting Galaxy into Multiple Processes
----------------------------------------

CloudMan's default configuration launches just one Galaxy process. To
really scale up the number of concurrent users Galaxy can handle it
must be split into multiple processes as outlined and documented `here
<http://wiki.galaxyproject.org/Admin/Config/Performance/Web%20Application%20Scal
ing>`.

     galaxy_conf_dir: /opt/galaxy/web/conf.d

     configure_multiple_galaxy_processes: True
     web_thread_count: 2
     handler_thread_count: 2

When these options are enabled, CloudMan will rewrite the body of the
``upstream galaxy_app {...}`` to load balance web traffic accross the
number of web threads you specify. However, when using multiple Galaxy
processes job admin functionality needs to be routed to Galaxy's job
manager, this can be done by updating your nginx.conf file to add the
following ``location /admin/jobs`` subsection shown below.


     location / {
         ...

         location /admin/jobs {
            proxy_pass  http://localhost:8079;
         }
     }



------------------------------
External Authentication (LDAP)
------------------------------

Galaxy requires a proxy web server (in our nginx) to enable external
authentication. See TODO:link_nginx for an example of how to enable
such authentication.

Galaxy also must be configured to use the information that will be
passed through by nginx. The method I have implemented for doing this,
involves telling CloudMan to use a Galaxy configuration directory
(TODO:link_to_pull_request) and then set the Galaxy options
use_remote_user, remote_user_maildomain, remote_user_logout_href, and
require_login appropriately.

galaxy_conf_dir: /opt/galaxy/web/conf.d
galaxy_universe_use_remote_user: True
galaxy_universe_remote_user_maildomain: <domain_name (e.g. example.org)>
galaxy_universe_remote_user_logout_href: https://logout@<galaxy_url>/
galaxy_universe_require_login: True

If you are using external authentication in this mannor it is also
likely a good idea to enable SSL.

----------
Enable SSL
----------

Your cloud security group will likely block port ``443`` by
default. This must be opened.

If you are using Amazon EC2, when following the instructions on the
`CloudMan <http://wiki.galaxyproject.org/CloudMan>` wiki site, be sure
to add the HTTPS inbound rule in addition to the HTTP one mentioned.

Instructions for opening this port on private clouds will vary, the
following command for instance will open it for OpenStack.

    nova secgroup-add-rule <security_group> tcp 443 443 0.0.0.0/0

CloudMan will need to setup the desired SSL key and cert before nginx
starts up. The CloudMan augmentations including 

    conf_files:
       - path: /usr/nginx/conf/key
         content: <base64 encoding of key>
       - path: /usr/nginx/conf/cert
         content: <base64 encoding of cert>

--------------
Reports Server
--------------

The Galaxy reports webapp is a small webapp that runs in parallel to
Galaxy and provides a wealth of valuable data on every job that Galaxy
has run as well as disk usage accounting, etc....

CloudMan can now enable the reports application by simply adding it to
the list of services.

    services:
      - name: Galaxy
      - name: GalaxyReports
      - name: Postgres

------------------------
External Postgres Server
------------------------

When deploying to Amazon, running a Postgres server right on the
CloudMan/Galaxy head node makes a lot of sense. But for private cloud
deployments, many institutions may already have well optimized, well
maintained production Postgres servers.

To disable the Postgres server, simply manually specify the list of
services CloudMan should start and exclude Postgres. For instance:

    services:
      - name: Galaxy
      - name: GalaxyReports

Then Galaxy must simply be configured to use your external postgres
server, this can be done by passing it in via the userdata variable
``galaxy_universe_database_connection``.

    galaxy_universe_database_connection: postgres://user:password@host:port/sche
ma


------------------------------------------
Running Jobs on External Compute Resources
------------------------------------------
