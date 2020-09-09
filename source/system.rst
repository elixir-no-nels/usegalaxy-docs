System
======

User management
---------------

There are two main users on the UseGalaxy.no infrastructure that is used
when connecting to the system (using ssh):

- sysadmin: The operating system manager.
- galaxyadmin: The Galaxy admin user. Have sudo access to the system user
  actually running the Galaxy processes.

Access to the accounts are managed using the vault encrypted file
`env/common/files/ssh/[sysadmin|galaxyadmin]/authorized_keys.vault`.

To implement any changes to these files, run the playbook inside both
the `env/main/` and `env/test/` folders:

.. code-block:: bash

    ansible-playbook system.yml --tags "users"
