
Creating a cloud instance the user data option ``galaxy_conf_dir`` will
instruct `CloudMan`_ to launch `Galaxy`_ with a environment variable
``GALAXY_UNIVERSE_CONFIG_DIR`` set to the value specified by
``galaxy_conf_dir``.

When `Galaxy`_ starts up with this set, it merges the files in the specified
directory into a single ``universe_wsgi.ini`` file. This allows configuration
by directory (like ``/etc/sudoers.d``).

Setting ``galaxy_conf_dir`` is required for many other options described in
this document - such as ``configure_multiple_galaxy_processes``,
``galaxy_config_*`` and ``galaxy_tool_runner_*``.
