Deployment of usegalaxy
=======================


The usegalaxy.no infrastructure is orchestrated by a set of ansible playbooks. These playbooks define and control
all software and configurations used by the usegalaxy.no setup.

There are three set of playbooks:

* main: this is the playbooks for the production infrastructure
* test: this is the playbooks for the test infrastructure
* common: things shared between the test and production sites. Generally these are symbolic links from main and test


In order to use these you will need the project ansible-vault password. If you don't have it, please send a request on XXXXXX.

Setting up a fresh repository with all the useglaxy.no infrastructure playbooks


Initial setup
-------------

.. code-block:: bash

  # clone the usegalaxy repository
  git clone git@github.com:usegalaxy-no/infrastructure-playbook.git

  cd infrastructure-playbook

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

  # deploy the database server
  ansible database.yml

  # deploy the galaxy frontend server with jenkins
  ansible galaxy.yml
  ansible jenkins.yml
  ansible tools.yml


  # deploy the local slurm compute node
  ansible slurm.yml

  # deploy the elastic nrec compute node
  ansible ehos.yml

  # deploy the orion cluster pulsar server
  ansible pulsar.yml



tools.yml
system.yml


