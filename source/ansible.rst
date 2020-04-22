Ansible documentation
=======

This covers the playbooks/tasks/roles used in the setting up and maintaining the usegalaxy.no infrastructure.


Playbooks
---------

The usegalaxy.no infrastructure consists of a set of playbooks addressing different servers



- database: setup the database server
- galaxy: setup the galaxy frontend server
- jenkins: configure the jenkins server on the galaxy frontend
- pulsar: The pulsar server on the nmbu compute
- ehos: the elastic compute on nrec
- slurm: configure the slurm nodes
- system: general boot strapping of all servers.


Tasks
-----

users_groups
^^^^^^^^^^^^

This task creates users and groups. If an authorized_keys file exists for a user this is furthermore copied across.

The tasks are controlled by two variables: system_groups and system_users. Both have a group_names that controls what
server_groups the user/group is created on. Only exception is all meaning that the user/group is created on all servers

The ssh-keys are encrypted with ansible-vault, and are located in files/ssh/<USERNAME>/authorized_keys.vault.

This task are using two additional tasks: _system_groups.yml and _system_users.yml


.. code-block:: yaml

  # will create the galaxy group (gid=1100) on the galaxyserver and slurm servers
  system_groups:
  - { name: galaxy, gid: 1100, group_names: ['galaxyserver', 'slurm'] }

.. code-block:: yaml

  # will create the sysadmin user (uid=1e00) on all servers
  system_users:
  - { name: sysadmin, uid: 1200, group: sysadmin, shell: /bin/bash, create_home: yes, group_names: ['all'] }



galaxy
^^^^^^

This task fulfill the following galaxy-frontend specific tasks:

- Add NeLS welcome page
- replaces defalt scss with the NeLS scss file (colours)
- rebuilding client
- Ensure compliance log exists
- configure gxadmin for galaxy user
- configure selinux, eg: allow nginx to access static




pulsar
^^^^^^

This task fulfill the following pulsar node specific tasks:

- cronjob to delete old job files
- daily restart of pulsar



rabbitmq
^^^^^^^^

The task are controlled by two variables: rabbitmq_vhosts and rabbitmq_users

.. code-block:: yaml

  rabbitmq_vhosts:
    - pulsar_test

.. code-block:: yaml

  rabbitmq_users:
    - {name: test_galaxy, vhost: pulsar_test,
       passwd: "{{test_rabbitmq_password}}"}



This task fulfill the following rabbitmq installation tweaks:
- delete guest user (hardening)
- create vhosts
- create users
- setup ampqs (ssl certs)
- apply bespoke config file


slurm
^^^^^

- copies across the slurm plugin for telegraf
