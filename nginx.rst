
``/usr/nginx/conf/nginx.conf`` is the main configuration used by the `nginx`_
web server that proxies traffic to `Galaxy`_. A custom ``nginx.conf``_  file
can be configured either while building the base image via `CloudBioLinux`_ or
while booting up the instance via `CloudMan`_.

\.\.\. via CloudMan
~~~~~~~~~~~~~~~~~~~~

`CloudMan`_ will replace ``nginx.conf`` with the base 64 encoded contents
specified by the user data option ``nginx_conf_contents``. 

::

    nginx_conf_contents: <base64 encoding of contents or URL>

One can use the \*nix utility `base64` to generate the base 64 encoding for a
file.

\.\.\. via CloudBioLinux
~~~~~~~~~~~~~~~~~~~~~~~~~

`CloudBioLinux_ contains default template for nginx.conf, but this can easily
`be substituted out for another file by setting the nginx_conf_path property
`to point to this new file in the target fabricrc_ file.

::

    nginx_conf_path = /local/path/to/nginx.conf

Alternatively the file can just be directly modified
`cloudbiolinux/installed_files/nginx.conf.template <https://github.com/chapmanb/cloudbiolinux/blob/master/installed_files/nginx.conf.template>`_ 
in the local clone of `CloudBioLinux`_. Be careful if directly editing this file however, this is treated as a template
so all `$` symbols must be escaped with a `\\` to avoid variable interpolation.