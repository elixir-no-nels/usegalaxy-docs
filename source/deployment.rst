Deployment of usegalaxy
=======================


The usegalaxy.no infrastructure is orchestrated by a set of ansible playbooks,
heavily inspired by the `Galaxy training project <https://training.galaxyproject.org>`__.
These playbooks define and control all software and configurations used by the usegalaxy.no setup.

There are three set of playbooks collections:

* main: this is the playbooks for the production infrastructure
* test: this is the playbooks for the test infrastructure
* common: things shared between the test and production sites. Generally these are symbolic links from main and test


In order to use these you will need the project ansible-vault password. If you don't have it, please send a request to admin(at)usegalaxy.no

Setting up a fresh repository with all the useglaxy.no infrastructure playbooks


Initial setup
-------------

.. code-block:: bash

  # clone the usegalaxy repository
  git clone git@github.com:usegalaxy-no/infrastructure-playbook.git

  cd infrastructure-playbook

  # setup the pre-commit hook that ensures no unencrypted vault-files
  # are commited
  cp pre-commit .git/hooks/
  chmod 755 .git/hooks/pre-commit
  

  # setup and activate a virtual environment
  python3 -m venv venv
  source venv/bin/activate

  # install python requirements
  pip install -r requirements.txt

  # install ansible requirements
  cd env/common
  ansible-galaxy install -p roles -r requirements.yml

  # Enter project password into vault_password
  $EDITOR vault_password



Deploying an infrastructure
---------------------------


.. code-block:: bash

  # Change into either main or test environment
  cd env/test

  # bootstrap the servers
  # As the nrec user is centos this needs to be run slightly differently
  ansible-playbook -e "ansible_user=centos" system.yml

  # deploy the database server
  ansible-playbook database.yml

  # deploy the local slurm compute node
  ansible-playbook slurm.yml

  # deploy the galaxy frontend server
  ansible-playbook  --extra-vars "__galaxy_dir_perms='0755'" galaxy.yml

  # deploy the elastic nrec compute node
  ansible-playbook ehos.yml

  # deploy the orion cluster pulsar server
  ansible-playbook pulsar.yml
  
  # deploy the cvmfs server
  ansible-playbook cvmfs-stratum0.yml

  # install tools 
  ansible-playbook tools.yml


tools.yml
system.yml


