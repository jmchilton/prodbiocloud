
You can specify a custom nginx.conf either while building the base image with
CloudBioLinux or while booting up the VM with CloudMan.

with CloudMan
-------------

To specify::

    nginx_conf_contents: <base64 encoding of contents or URL>

One can use the *nix utility ``base64`` to generate the base 64 encoding for a
file.

with CloudBioLinux
------------------

Update your fabricrc file with:: 

    nginx_conf_path = /local/path/to/nginx.conf

Alternatively one can just directly edit
``cloudbiolinux/installed_files/nginx.conf.template`` in the local clone of
CloudBioLinux. Be careful though, this is a template not the direct contents
so you must escape all ``$`` symbols with a ``\`` to avoid variable
interpolation.
