
Creating a cloud instance the user data option ``galaxy_conf_dir`` will
instruct `CloudMan`_ to launch `Galaxy`_ with the environment variable
``GALAXY_UNIVERSE_CONFIG_DIR`` set to the value specified by
``galaxy_conf_dir``.

Since `pull request 44 <https://bitbucket.org/galaxy/galaxy-central/pull-request/44/allow-usage-of-directory-of-configuration>`_,  when `Galaxy`_
starts up with this set, it merges the files in the specified directory into a
single ``universe_wsgi.ini`` file. This allows configuration by directory
(like the ``/etc/sudoers.d`` or ``/etc/apache/conf.d`` directories in many
modern Linux distributions).

Setting ``galaxy_conf_dir`` eases `CloudMan`_'s ability to configure `Galaxy`_
and is required for many other options described in this document - such as
``configure_multiple_galaxy_processes``, ``galaxy_config_*`` and
``galaxy_tool_runner_*``.
